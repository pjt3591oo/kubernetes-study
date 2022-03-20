# minikube에서 로컬 이미지 띄울때

* react 어플리케이션

```bash
$ npx create-react-app react-nginx-app
```

* Dockerfile 작성

```Dockerfile
# ========== 빌드 환경 ==========

# 베이스 이미지 선택
FROM node:alpine as builder

# 작업 디렉터리 지정
WORKDIR '/app'

# 리액트 프로젝트 이미지 안으로 복사
COPY react-nginx-app/ .

# 의존성 모듈 설치
RUN npm install

# 빌드
RUN npm run build

# ========== 프로덕션 환경 ==========
FROM nginx

# 빌드환경에서 빌드 결과를 프로덕션 환경인 nginx 안으로 복사
COPY --from=builder /app/build /usr/share/nginx/html
```

* 이미지 빌드

```
$ docker build -t react/nginx .
```

* pod 정의

```yaml
apiVersion: v1
kind: Pod
metadata:
 name: react-nginx
 labels:
   app: react-nginx-app
   pod-template-hash: ff6768df4
spec:
  containers:
  - name: react-nginx-app
    image: react/nginx:0.1
    ports:
    - containerPort: 80
```

* pod 실행

```bash
$ kubectl apply -f test.pod.local.image.yaml
```

* pod 조회

```bash
$ kubectl get pod -A

NAME          READY   STATUS         RESTARTS   AGE
react-nginx   0/1     ErrImagePull   0          16s
```

```bash
$ kubectl describe pod react-nginx

Events:
  Type     Reason     Age                From               Message
  ----     ------     ----               ----               -------
  Normal   Scheduled  93s                default-scheduler  Successfully assigned default/react-nginx to minikube
  Warning  Failed     45s (x3 over 89s)  kubelet            Failed to pull image "react/nginx:0.1": rpc error: code = Unknown desc = Error response from daemon: pull access denied for react/nginx, repository does not exist or may require 'docker login': denied: requested access to the resource is denied
  Warning  Failed     45s (x3 over 89s)  kubelet            Error: ErrImagePull
  Normal   BackOff    16s (x4 over 88s)  kubelet            Back-off pulling image "react/nginx:0.1"
  Warning  Failed     16s (x4 over 88s)  kubelet            Error: ImagePullBackOff
  Normal   Pulling    2s (x4 over 92s)   kubelet            Pulling image "react/nginx:0.1"
```

* event 확인

```bash
$ kubectl get events

LAST SEEN   TYPE      REASON                    OBJECT            MESSAGE
42m         Normal    NodeHasSufficientMemory   node/minikube     Node minikube status is now: NodeHasSufficientMemory
42m         Normal    NodeHasNoDiskPressure     node/minikube     Node minikube status is now: NodeHasNoDiskPressure
42m         Normal    NodeHasSufficientPID      node/minikube     Node minikube status is now: NodeHasSufficientPID
42m         Normal    Starting                  node/minikube     Starting kubelet.
42m         Normal    NodeHasSufficientMemory   node/minikube     Node minikube status is now: NodeHasSufficientMemory
42m         Normal    NodeHasNoDiskPressure     node/minikube     Node minikube status is now: NodeHasNoDiskPressure
42m         Normal    NodeHasSufficientPID      node/minikube     Node minikube status is now: NodeHasSufficientPID
42m         Normal    NodeNotReady              node/minikube     Node minikube status is now: NodeNotReady
42m         Normal    NodeAllocatableEnforced   node/minikube     Updated Node Allocatable limit across pods
42m         Normal    NodeReady                 node/minikube     Node minikube status is now: NodeReady
42m         Normal    RegisteredNode            node/minikube     Node minikube event: Registered Node minikube in Controller
42m         Normal    Starting                  node/minikube     Starting kube-proxy.
9m23s       Normal    Scheduled                 pod/react-nginx   Successfully assigned default/react-nginx to minikube
7m52s       Normal    Pulling                   pod/react-nginx   Pulling image "react/nginx:0.1"
7m48s       Warning   Failed                    pod/react-nginx   Failed to pull image "react/nginx:0.1": rpc error: code = Unknown desc = Error response from daemon: pull access denied for react/nginx, repository does not exist or may require 'docker login': denied: requested access to the resource is denied
7m48s       Warning   Failed                    pod/react-nginx   Error: ErrImagePull
4m16s       Normal    BackOff                   pod/react-nginx   Back-off pulling image "react/nginx:0.1"
7m23s       Warning   Failed                    pod/react-nginx   Error: ImagePullBackOff
```

event를 통해서도 pod 상태를 확인할 수 있다.

### 에러 발생 이유

해당 에러가 발생하는 이유는 minikube가 관리하는 이미지 레포지토리에 이미지가 존재하지 않기 때문이다.

```bash
$ minikube ssh
```

```bash
docker@minikube:~$ docker images
REPOSITORY                                TAG        IMAGE ID       CREATED         SIZE
k8s.gcr.io/kube-apiserver                 v1.21.7    0f5bfd20d26e   4 months ago    126MB
k8s.gcr.io/kube-controller-manager        v1.21.7    7a37590177f7   4 months ago    120MB
k8s.gcr.io/kube-proxy                     v1.21.7    7e58936d778d   4 months ago    104MB
k8s.gcr.io/kube-scheduler                 v1.21.7    c67c2461177d   4 months ago    50.9MB
kubernetesui/dashboard                    v2.3.1     e1482a24335a   9 months ago    220MB
kubernetesui/metrics-scraper              v1.0.7     7801cfc6d5c0   9 months ago    34.4MB
gcr.io/k8s-minikube/storage-provisioner   v5         6e38f40d628d   11 months ago   31.5MB
k8s.gcr.io/pause                          3.4.1      0f8457a4c2ec   14 months ago   683kB
k8s.gcr.io/coredns/coredns                v1.8.0     296a6d5035e2   17 months ago   42.5MB
k8s.gcr.io/etcd                           3.4.13-0   0369cf4303ff   18 months ago   253MB
```

앞에서 빌드한 이미지는 호스트에 존재하는 도커 이미지 레포지토리에 저장되는 반면에 minikube는 도커의 컨테이너이므로 minikube 컨테이너 레포지토리에 저장되지 않는다.

그렇다면 앞에서 만든 이미지를 minikube 이미지 레포지토리에도 만들어주면 되는것 아닌가? 

그렇다!

* 해결방법

minikube의 이미지 레포지토리로 이미지를 넘기는 방법은 8가지가 있다고 한다. 그 중 하나가 다음 명령어를 통해 이미지를 넘길 수 있다.

```bash
$ minikube image load react/nginx:0.1
```

약 10초 정도 시간이 지나면 이미지를 넘길 수 있다.

```bash
$ minikube ssh
```

```bash
docker@minikube:~$ docker images
REPOSITORY                                TAG        IMAGE ID       CREATED         SIZE
react/nginx                               0.1        b3b1973a2227   3 hours ago     142MB
k8s.gcr.io/kube-apiserver                 v1.21.7    0f5bfd20d26e   4 months ago    126MB
k8s.gcr.io/kube-controller-manager        v1.21.7    7a37590177f7   4 months ago    120MB
k8s.gcr.io/kube-proxy                     v1.21.7    7e58936d778d   4 months ago    104MB
k8s.gcr.io/kube-scheduler                 v1.21.7    c67c2461177d   4 months ago    50.9MB
kubernetesui/dashboard                    v2.3.1     e1482a24335a   9 months ago    220MB
kubernetesui/metrics-scraper              v1.0.7     7801cfc6d5c0   9 months ago    34.4MB
gcr.io/k8s-minikube/storage-provisioner   v5         6e38f40d628d   11 months ago   31.5MB
k8s.gcr.io/pause                          3.4.1      0f8457a4c2ec   14 months ago   683kB
k8s.gcr.io/coredns/coredns                v1.8.0     296a6d5035e2   17 months ago   42.5MB
k8s.gcr.io/etcd                           3.4.13-0   0369cf4303ff   18 months ago   253MB
```

* pods 상태조회

```bash
$ kubectl get pods
NAME          READY   STATUS    RESTARTS   AGE
react-nginx   1/1     Running   0          7m34s
```

pod은 정상적으로 동작한다.