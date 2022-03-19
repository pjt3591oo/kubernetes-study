### deployment 

* deployment 배포

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: test-deployment   # deployment의 이름
  labels:
    app: test-deployment  # deployment의 라벨
spec:
  replicas: 3
  selector:
    matchLabels:
      app: test-deployment # replicaset이 관리할 pod 라벨
  template:    # 하위에 pod 정보 설정
    metadata:
      labels:
        app: test-deployment # pod의 라벨
    spec: # 컨테이너 설정
      containers:
      - name: test-deployment
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

test-deployment 이름을 가진 deployment 생성

해당 deployment는 test-deployment 라벨을 가진 3개의 pod을 관리하는 replicaset을 관리함

```bash
$ kubectl apply -f test.deployment.yaml

deployment.apps/test-deployment created
```

* deployment 조회

```bash
$ kubectl get deployment

NAME              READY   UP-TO-DATE   AVAILABLE   AGE
test-deployment   3/3     3            3           2m6s
```

* pod 조회

```bash
$ kubectl get pods

NAME                               READY   STATUS    RESTARTS   AGE
test-deployment-7486d94ddd-6n2f7   1/1     Running   0          2m10s
test-deployment-7486d94ddd-g6ppb   1/1     Running   0          2m10s
test-deployment-7486d94ddd-xfnt9   1/1     Running   0          2m10s
```

* pod 삭제

```bash
$ kubectl delete pod test-deployment-7486d94ddd-6n2f7

pod "test-deployment-7486d94ddd-6n2f7" deleted
```

replica-set은 pod 개수가 줄었기 때문에 3개를 유지하기 위해 1개의 pod을 생성한다.

```
$ kubectl get pods                                         

NAME                               READY   STATUS    RESTARTS   AGE
test-deployment-7486d94ddd-g6ppb   1/1     Running   0          19m
test-deployment-7486d94ddd-r2rdf   1/1     Running   0          27s
test-deployment-7486d94ddd-xfnt9   1/1     Running   0          19m
```

* deployment 업데이트

nginx:1.7.9 -> nginx:1.9.1

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: test-deployment   # deployment의 이름
  labels:
    app: test-deployment  # deployment의 라벨
spec:
  replicas: 3
  selector:
    matchLabels:
      app: test-deployment # replicaset이 관리할 pod 라벨
  template:    # 하위에 pod 정보 설정
    metadata:
      labels:
        app: test-deployment # pod의 라벨
    spec: # 컨테이너 설정
      containers:
      - name: test-deployment
        image: nginx:1.9.1
        ports:
        - containerPort: 80
```

apply를 이용하여 적용한다.

```bash
$ kubectl apply -f test.deployment.yaml

deployment.apps/test-deployment configured
```

```bash
$ kubectl get deployments                                  

NAME              READY   UP-TO-DATE   AVAILABLE   AGE
test-deployment   3/3     1            3           22m
```

pod을 조회하면 다음과 같이 ContainerCreating의 상태를 가진 pod을 확인할 수 있다.

```bash
$ kubectl get pods                                         

NAME                               READY   STATUS              RESTARTS   AGE
test-deployment-7486d94ddd-g6ppb   1/1     Running             0          22m
test-deployment-7486d94ddd-r2rdf   1/1     Running             0          3m20s
test-deployment-7486d94ddd-xfnt9   1/1     Running             0          22m
test-deployment-ff6768df4-qrdlt    0/1     ContainerCreating   0          14s
```

시간이 조금 지난후 pod을 조회하면 이미지가 업데이트 된 pod이 생성된 모습을 확인할 수 있다.

```bash
$ kubectl get pods

NAME                              READY   STATUS    RESTARTS   AGE
test-deployment-ff6768df4-jrt6s   1/1     Running   0          39s
test-deployment-ff6768df4-qrdlt   1/1     Running   0          61s
test-deployment-ff6768df4-zbgxk   1/1     Running   0          42s
```

* deployment 상세내용 조회

```bash
$ kubectl describe deployment test-deployment

Name:                   test-deployment
Namespace:              default
CreationTimestamp:      Wed, 16 Mar 2022 21:47:01 +0900
Labels:                 app=test-deployment
Annotations:            deployment.kubernetes.io/revision: 2
Selector:               app=test-deployment
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=test-deployment
  Containers:
   test-deployment:
    Image:        nginx:1.9.1
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   test-deployment-ff6768df4 (3/3 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  25m    deployment-controller  Scaled up replica set test-deployment-7486d94ddd to 3
  Normal  ScalingReplicaSet  3m26s  deployment-controller  Scaled up replica set test-deployment-ff6768df4 to 1
  Normal  ScalingReplicaSet  3m7s   deployment-controller  Scaled down replica set test-deployment-7486d94ddd to 2
  Normal  ScalingReplicaSet  3m7s   deployment-controller  Scaled up replica set test-deployment-ff6768df4 to 2
  Normal  ScalingReplicaSet  3m4s   deployment-controller  Scaled down replica set test-deployment-7486d94ddd to 1
  Normal  ScalingReplicaSet  3m4s   deployment-controller  Scaled up replica set test-deployment-ff6768df4 to 3
  Normal  ScalingReplicaSet  3m3s   deployment-controller  Scaled down replica set test-deployment-7486d94ddd to 0
```

* deployment history 조회

```bash
$ kubectl rollout history deployment test-deployment

deployment.apps/test-deployment
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

rollout history에서 CHANGE_CAUSE가 <none>이 나온 이유는 .yaml에 metadata.annotations에 kubernetes.io/change-cause: [기록할 단어]를 입력하면 된다. 일반적으로 버전을 입력하면 좋을 것 같다.

* replicaset 조회

```bash
$ kubectl get replicaset                                      

NAME                         DESIRED   CURRENT   READY   AGE
test-deployment-7486d94ddd   0         0         0       29m
test-deployment-ff6768df4    3         3         3       7m18s
```

기존의 replicaset의 scale은 0으로 감소되고 새로운 replicaset을 생성한다.

* 롤백

```bash
$ kubectl rollout undo deployment test-deployment --to-revision=1

deployment.apps/test-deployment rolled back
```

--to-revision은 rollout history 시에 조회된 돌아갈 REVISION을 명시한다.

describe를 조회하면 nginx:1.7.9로 돌아간 모습을 확인할 수 있다. 또한 replicaset을 조회하면 기존에 동작하던 replicaset이 동작하는 모습을 볼 수 있다.

```bash
$ kubectl get replicaset                                      

NAME                         DESIRED   CURRENT   READY   AGE
test-deployment-7486d94ddd   3         3         3       37m
test-deployment-ff6768df4    0         0         0       14m
```

* CHANGE_CAUSE 기록하기

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: test-deployment   # deployment의 이름
  labels:
    app: test-deployment  # deployment의 라벨
  annotations: 
    kubernetes.io/change-cause: version1.9.1
spec:
  replicas: 3
  selector:
    matchLabels:
      app: test-deployment # replicaset이 관리할 pod 라벨
  template:    # 하위에 pod 정보 설정
    metadata:
      labels:
        app: test-deployment # pod의 라벨
    spec: # 컨테이너 설정
      containers:
      - name: test-deployment
        image: nginx:1.9.1
        ports:
        - containerPort: 80
```

```bash
$ kubectl apply -f test.deployment.yaml
```

annotations가 반영된 yaml을 기반으로 deployment를 재배포 한다

```bash
$ kubectl rollout history deployment test-deployment

deployment.apps/test-deployment
REVISION  CHANGE-CAUSE
3         <none>
4         version1.9.1
```

* 중지, 시작, 재시작

```bash
# 배포 일시 중지
$ kubectl rollout pause deployment test-deployment

# 배포 시작
$ kubectl rollout resume deployment test-deployment

# 배포 재시작
$ kubectl rollout restart deployment test-deployment
```

* pod 생성

replica-set은 라벨 단위로 pod을 관리하기 때문에 동일한 라벨을 가진 pod이 생성되면 기존의 pod이 제거될 수 있다.

```bash
$ kubectl get pods --show-labels

NAME                              READY   STATUS    RESTARTS   AGE     LABELS
test-deployment-ff6768df4-h622p   1/1     Running   0          4m24s   app=test-deployment,pod-template-hash=ff6768df4
test-deployment-ff6768df4-jpdfj   1/1     Running   0          4m23s   app=test-deployment,pod-template-hash=ff6768df4
test-deployment-ff6768df4-kz49q   1/1     Running   0          4m27s   app=test-deployment,pod-template-hash=ff6768df4
```

동일한 라벨(test-deployment)을 가진 pods를 생성하기 전에 기존에 생성된 pods 확인

```yaml
apiVersion: v1
kind: Pod
metadata:
 name: myapp-pod
 labels:
   app: test-deployment
   pod-template-hash: ff6768df4
spec:
  containers:
  - name: test-deployment
    image: nginx:1.7.9
    ports:
    - containerPort: 80
```

여기서 핵심은 동일한 라벨을 설정하는 것!!

deployment로 생성된 pod의 라벨과 완전히 동일하게 한다.

```bash
$ kubectl apply -f test.pod.yaml

pod/myapp-pod created
```

```bash
$ kubectl get pods --show-labels

NAME                              READY   STATUS        RESTARTS   AGE    LABELS
myapp-pod                         0/1     Terminating   0          3s     app=test-deployment,pod-template-hash=ff6768df4
test-deployment-ff6768df4-h622p   1/1     Running       0          8m2s   app=test-deployment,pod-template-hash=ff6768df4
test-deployment-ff6768df4-jpdfj   1/1     Running       0          8m1s   app=test-deployment,pod-template-hash=ff6768df4
test-deployment-ff6768df4-kz49q   1/1     Running       0          8m5s   app=test-deployment,pod-template-hash=ff6768df4
```

여기선 현재 생성한 pod가 Terminating(종료) 되었지만 deployment에서 생성된 pod가 Terminating될 수 있다.

* 로그보기

```bash
$ kubectl logs test-deployment-ff6768df4-h622p
```

* 컨테이너 접속

```bash
$ kubectl exec -it test-deployment-ff6768df4-h622p -- /bin/sh
```

*  포트포워딩

```bash
$ kubectl port-forward test-deployment-ff6768df4-h622p 8080:80

Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
```

호스트 8080번 포트로 접속한 트래픽을 해당 pod의 80번 포트로 포워딩한다.

```bash
$ curl http://localhost:8080

<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

이제 해당 pod의 로그를 보면 접속 로그를 확인할 수 있다.

```bash
$ kubectl logs test-deployment-ff6768df4-h622p

127.0.0.1 - - [16/Mar/2022:13:41:28 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.64.1" "-"
```

포트포워딩은 특정 pod으로만 접속을 허용한다. 만약 로드벨런싱 기능을 하기 위해선 서비스를 활용한다.

### 서비스

deployment는 replicaset을 통해 pod을 관리한다. deployment로 생성된 pod은 자체적으로 통신할 수단이 존재하지 않는다.

서비스를 통해 deployment로 생성된 pod이 외부 서비스와 통신할 수 있도록 할 수 있다.

* 서비스 정의

서비스는 4개의 타입이 존재함

  - ClusterIP
  - NodePort
  - LoadBalancer
  - ExternalName

```yaml
apiVersion: v1
kind: Service
metadata:
  name: test-svc
spec:
  type: NodePort
  selector:
    app: test-deployment
  ports:
    - name: http
      port: 8080
      protocol: TCP
      targetPort: 80  # pod에 실행중인 어플리케이션의 포트(nginx가 80포트로 running중)
      nodePort: 31000
    - name: https
      port: 443
      protocol: TCP
      targetPort: 80 # pod에 실행중인 어플리케이션의 포트(nginx가 80포트로 running중)
      nodePort: 31001
```

```bash
$ kubectl apply test.service.yaml

service/test-svc created
```

* IP 확인

minikube에서 서비스 동작을 확인하기 위해 localhost:31000으로 요청

```
$ minikube ip
```

```bash
$ minikube ssh # docker exec -it minikube

docker@minikube:~$ curl localhost:31000

<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

curl을 이용하여 여러번 요청해본다.

* 로그확인

```bash
$  kubectl logs --namespace default -l app=test-deployment --timestamps=true --prefix=true --all-containers=true

[pod/test-deployment-ff6768df4-8nsjv/test-deployment] 172.17.0.1 - - [18/Mar/2022:23:30:40 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.68.0" "-"
[pod/test-deployment-ff6768df4-8nsjv/test-deployment] 172.17.0.1 - - [18/Mar/2022:23:30:41 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.68.0" "-"
[pod/test-deployment-ff6768df4-8nsjv/test-deployment] 172.17.0.1 - - [18/Mar/2022:23:30:41 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.68.0" "-"
[pod/test-deployment-ff6768df4-8nsjv/test-deployment] 172.17.0.1 - - [18/Mar/2022:23:31:31 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.68.0" "-"
[pod/test-deployment-ff6768df4-8nsjv/test-deployment] 172.17.0.1 - - [18/Mar/2022:23:31:32 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.68.0" "-"
[pod/test-deployment-ff6768df4-hj77f/test-deployment] 172.17.0.1 - - [18/Mar/2022:23:31:29 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.68.0" "-"
[pod/test-deployment-ff6768df4-hj77f/test-deployment] 172.17.0.1 - - [18/Mar/2022:23:31:29 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.68.0" "-"
[pod/test-deployment-ff6768df4-hj77f/test-deployment] 172.17.0.1 - - [18/Mar/2022:23:31:30 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.68.0" "-"
[pod/test-deployment-ff6768df4-hj77f/test-deployment] 172.17.0.1 - - [18/Mar/2022:23:31:31 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.68.0" "-"
[pod/test-deployment-ff6768df4-hj77f/test-deployment] 172.17.0.1 - - [18/Mar/2022:23:31:31 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.68.0" "-"
[pod/test-deployment-ff6768df4-hj77f/test-deployment] 172.17.0.1 - - [18/Mar/2022:23:31:32 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.68.0" "-"
[pod/test-deployment-ff6768df4-hj77f/test-deployment] 172.17.0.1 - - [18/Mar/2022:23:33:06 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.68.0" "-"
[pod/test-deployment-ff6768df4-hj77f/test-deployment] 172.17.0.1 - - [18/Mar/2022:23:33:07 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.68.0" "-"
[pod/test-deployment-ff6768df4-hj77f/test-deployment] 172.17.0.1 - - [18/Mar/2022:23:33:07 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.68.0" "-"
[pod/test-deployment-ff6768df4-hj77f/test-deployment] 172.17.0.1 - - [18/Mar/2022:23:33:08 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.68.0" "-"
[pod/test-deployment-ff6768df4-xrpl9/test-deployment] 172.17.0.1 - - [18/Mar/2022:23:30:38 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.68.0" "-"
[pod/test-deployment-ff6768df4-xrpl9/test-deployment] 172.17.0.1 - - [18/Mar/2022:23:30:39 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.68.0" "-"
[pod/test-deployment-ff6768df4-xrpl9/test-deployment] 172.17.0.1 - - [18/Mar/2022:23:30:41 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.68.0" "-"
[pod/test-deployment-ff6768df4-xrpl9/test-deployment] 172.17.0.1 - - [18/Mar/2022:23:31:32 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.68.0" "-"
[pod/test-deployment-ff6768df4-xrpl9/test-deployment] 172.17.0.1 - - [18/Mar/2022:23:31:32 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.68.0" "-"
[pod/test-deployment-ff6768df4-xrpl9/test-deployment] 172.17.0.1 - - [18/Mar/2022:23:31:32 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.68.0" "-"
[pod/test-deployment-ff6768df4-xrpl9/test-deployment] 172.17.0.1 - - [18/Mar/2022:23:33:06 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.68.0" "-"
```

서비스에 의해 적절히 로드벨런싱이 이루어진 모습을 확인할 수 있다.

### 인그레스(ingress)

서비스는 팟과 통신하기 위한 방법을 정의하는 반면에 인그레스는 클러스터 외부에서 내부로 접근하는 방법을 정의합니다.

인그레스를 사용하기 위해 ingress, ingress-controller, ingress-nginx 적용 필요

* 인그레스 활성화

minikube는 다음 명령어로 nginx ingress controller를 활성화할 수 있다.

```bash
$ minikube addons enable ingress
```

* 인그레스 컨트롤러 확인

```bash
$ kubectl -n ingress-nginx get pod

NAME                                        READY   STATUS      RESTARTS   AGE
ingress-nginx-admission-create-8ljk5        0/1     Completed   0          23m
ingress-nginx-admission-patch-n6cbj         0/1     Completed   0          23m
ingress-nginx-controller-5fc9586f46-wrsg7   1/1     Running     0          23m
```

```bash
$ kubectl get svc -n ingress-nginx

NAME                                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             NodePort    10.107.8.147    <none>        80:31922/TCP,443:30248/TCP   23m
ingress-nginx-controller-admission   ClusterIP   10.109.78.124   <none>        443/TCP                      23m
```

```bash
$ minikube ssh # docker exec -it minikube

docker@minikube:~$ curl -I http://localhost/healthz
HTTP/1.1 200 OK
Date: Sat, 19 Mar 2022 00:39:22 GMT
Content-Type: text/html
Content-Length: 0
Connection: keep-alive
```

```bash
$ minukube addons list

|-----------------------------|----------|--------------|--------------------------------|
|         ADDON NAME          | PROFILE  |    STATUS    |           MAINTAINER           |
|-----------------------------|----------|--------------|--------------------------------|
| ambassador                  | minikube | disabled     | third-party (ambassador)       |
| auto-pause                  | minikube | disabled     | google                         |
| csi-hostpath-driver         | minikube | disabled     | kubernetes                     |
| dashboard                   | minikube | disabled     | kubernetes                     |
| default-storageclass        | minikube | enabled ✅   | kubernetes                     |
| efk                         | minikube | disabled     | third-party (elastic)          |
| freshpod                    | minikube | disabled     | google                         |
| gcp-auth                    | minikube | disabled     | google                         |
| gvisor                      | minikube | disabled     | google                         |
| helm-tiller                 | minikube | disabled     | third-party (helm)             |
| ingress                     | minikube | enabled ✅   | unknown (third-party)          |
| ingress-dns                 | minikube | disabled     | google                         |
| istio                       | minikube | disabled     | third-party (istio)            |
| istio-provisioner           | minikube | disabled     | third-party (istio)            |
| kong                        | minikube | disabled     | third-party (Kong HQ)          |
| kubevirt                    | minikube | disabled     | third-party (kubevirt)         |
| logviewer                   | minikube | disabled     | unknown (third-party)          |
| metallb                     | minikube | disabled     | third-party (metallb)          |
| metrics-server              | minikube | disabled     | kubernetes                     |
| nvidia-driver-installer     | minikube | disabled     | google                         |
| nvidia-gpu-device-plugin    | minikube | disabled     | third-party (nvidia)           |
| olm                         | minikube | disabled     | third-party (operator          |
|                             |          |              | framework)                     |
| pod-security-policy         | minikube | disabled     | unknown (third-party)          |
| portainer                   | minikube | disabled     | portainer.io                   |
| registry                    | minikube | disabled     | google                         |
| registry-aliases            | minikube | disabled     | unknown (third-party)          |
| registry-creds              | minikube | disabled     | third-party (upmc enterprises) |
| storage-provisioner         | minikube | enabled ✅   | google                         |
| storage-provisioner-gluster | minikube | disabled     | unknown (third-party)          |
| volumesnapshots             | minikube | disabled     | kubernetes                     |
|-----------------------------|----------|--------------|--------------------------------|
```

* 인그레스 정의

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: my-ingress
  # namespace: namespace
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    kubernetes.io/ingress.class: nginx
spec:
  rules:
  # - host: mung.com # host를 명시하지 않으면 IPfh 연결
  - http:
      paths:
      - path: /
        backend:
          serviceName: test-svc
          servicePort: 8080
```

/으로 요청이 들어오면 test-svc로 정의된 서비스의 8080포트로 연결해준다.

```bash
$ kubectl apply -f test.ingress.yaml

ingress.extensions/my-ingress created
```

```bash
$ kubectl get ingress

NAME         CLASS    HOSTS   ADDRESS        PORTS   AGE
my-ingress   <none>   *       192.168.49.2   80      2m23s
```

address가 표시되는데 1~2분 정도 소요될 수 있다. 또한 ingress는 외부에서 접속하는 목적을 가지기 때문에 ADDRESS는 해당 노드의 아이피로 바인딩 된다.

```bash
$ kubectl get endpoints

NAME         ENDPOINTS                                               AGE
kubernetes   192.168.49.2:8443                                       25m
test-svc     172.17.0.4:80,172.17.0.5:80,172.17.0.6:80 + 3 more...   20m
```

여기서 kubernetes는 도커 기반으로 생성된 minikube에게 할당된 아이피이다.

```bash
$ minikube profile list

|----------|-----------|---------|--------------|------|---------|---------|-------|
| Profile  | VM Driver | Runtime |      IP      | Port | Version | Status  | Nodes |
|----------|-----------|---------|--------------|------|---------|---------|-------|
| minikube | docker    | docker  | 192.168.49.2 | 8443 | v1.21.7 | Running |     1 |
|----------|-----------|---------|--------------|------|---------|---------|-------|
```

```bash
$ minikube ssh # docker exec -it minikube

docker@minikube:~$ curl localhost
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

### 터널링

```bash
$ minikube ip
```

```bash
$ minikube ssh # docker exec -it minikube
```

기본적으로 도커 컨테이너로 생성된 minikube가 쿠버네티스의 마스터 노드로 작동하기 때문에 ingress, service를 사용하더라도 해당 노드에 접속한 후 설정된 아이피로 접속을 수행하였다.

만약, 해당 컨테이너(노드)에 접속하지 않고 인그레스로 설정된 아이피:포트로 접속하고 싶다면 터널링을 사용한다.

```bash
$ minikube tunnel

✅  Tunnel successfully started

📌  NOTE: Please do not close this terminal as this process must stay alive for the tunnel to be accessible ...

❗  The service/ingress my-ingress requires privileged ports to be exposed: [80 443]
🔑  sudo permission will be asked for it.
🏃  my-ingress 서비스의 터널을 시작하는 중
Password:
```

비밀번호를 입력하면 터널링이 동작한다. 이제 minikube ssh가 아닌 호스트에서 localhost로 접속하면 인그레스 - 서비스 - 팟에 접속할 수 있다. 터널링은 80, 443을 사용할 수 있다.

```bash
$ curl localhost
```

또는 크롬과 같은 브라우저를 통해 접속해보자.