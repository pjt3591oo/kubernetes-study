### 기본용어

* 마스터(Master): 노드를 제어하고 전체적인 클러스터를 관리해주는 컨트롤러, 전체적인 제어/관리하기 위한 관리 서버

* 노드(Node): 컨테이너가 배포될 물리 서버 또는 가상 머신, 워커 노드(Worker Node)라고도 함.

* 파드(Pod): 단일 노드에 배포된 하나 이상의 컨테이너 그룹. 파드라는 단위로 여러개의 컨테이너를 묶어서 파드 단위로 관리할 수 있음

### 쿠버네티스 컴포넌트

##### 마스터

* kube-apiserver

* etcd

* kube-scheduler

* kube-controller-manager

  * node controller

  * replication controller

  * endpoint controller

  * serice account & Token controller

* cloud-controller-manager

  * node controller

  * route controller

  * service controller

  * volume controller

##### 노드

* kubelet

* kube-proxy

* container runtime

##### 애드온

* dashboard

* container resource monitering 

* cluster logging

### 컨트롤러

* 데몬 셋(DaemonSet): 

* 스테이트풀 셋(StatefulSet):

* 디플로이먼트(Deployment):

* 레플리카셋(ReplicaSet):

* 잡(Job): 

### 터미널 사용법

[여기에 있어요~](./command.md)