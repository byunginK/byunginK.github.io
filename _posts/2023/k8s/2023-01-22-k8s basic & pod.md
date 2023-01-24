---
layout: post
title: "kubernetes 기초 1 (pod)"
date: 2023-01-22
categories: [k8s]
---

# k8s 설치

1. 각 노드에 CRI를 설치
2. swap 을 diable한다. `sudo swapoff -a`
3. 네트워크 bridge 와 필수 리스너 포트 설정

```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# sysctl params required by setup, params persist across reboots
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system
```

4. kubelet, kubeadm, kubectl 설치 (공홈 참고)
5. control-plane 구성
6. control-plane에 `kubeadm init`으로 마스터 노드 구성
7. control-plane에 출력된 token을 기억
8. kubectl을 사용할 수 있도록 아래 명령어로 추가

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

9. CNI를 설치하여 네트워크 구성
10. worker 노드와 master join 해준다.(아까 token이 포함된 명령어를 node에서 실행)

# kubectl 명령어 기본 구조

### kubectl [command] [TYPE][name][flags]

- cmmand : 자원에 실행할 명령 (create, get ,delete, edit)
- type : 자원의 타입 (node, pod, service ...)
- name : 자원의 이름
- flags : 부가적 설정할 옵션 (--help, -o options)

## image로 yaml 파일 만드는법

`kubectl run [pod name] --image=[image 이름 및 버전] --dry-run -o yaml > [파일이름].yaml`
dry-run은 실제 실행이 아닌 테스트 실행 느낌

## 쿠버네티스 컴포넌트

### 1. 마스터 컴포넌트

- etcd : key-value 타입의 저장소
- kube-apiserver : k8s api를 사용하도록 요청을 받고 요청이 유효한지 검사
- kube-scheduler : 파드를 실행할 노드 선택
- kube-controller-manager : 파드를 관찰하며 개수를 보장

### 2. 워커 노드 컴포넌트

- kubelet : 모든 노드에서 실행되는 k8s 에이전트 , 데몬형태로 동작
- kube-proxy : k8s의 network 동작을 관리, iptables rule을 구성
- 컨테이너 런타임 : 컨테이너를 실행하는 엔진 (docker, containerd, runc)

### 3. 애드온

- 네트워크 애드온 (CNI: weace, calico, flanedld, kube-route...)
- dns 애드온 (core DNS)
- 대시보드 애드온
- 컨테이너 자원 모니터링 (cAdvisor)
- 클러스터 로깅 (컨테이너 로그, k8s 운영로그 중앙화[ELK, EFK datadog])

### k8s namespace

클러스터 하나를 여러개의 논리적인 단위로 나눠서 사용 (용도에 따라 실행해야 하는 앱을 구분할때 사용)

### API 버전

- k8s object 정의 시 api version이 필요
- k8s가 update하는 api가 있으면 새로운 api가 생성됨
- yaml파일에 apiVersion에 각 api의 종류에 따라 버전을 알맞게 입력해야한다.

##### API Object의 종류 및 버전

- 각 api 의 버전 확인 방법은 explain 명령어를 통해 확인 가능하다. pod 버전 확인 예:`kubectl explain pod`

## Pod 란

컨테이너를 표현하는 k8s API의 최소단위

### pod 생성

CLI, pod yaml을 생성해서 생성 `kubectl create -f [yaml파일 명]`

##### yaml 예시

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: [pod 이름]
spec:
  containers:
    - name: nginx-container (container 이름)
      image: nginx:1.14
      imagePullPolicy: Always
      ports:
        - containerPotr: 80
          protocol: TCP
```

### Pod 확인

```
$ kubectl get pods
$ kubectl get pods -o wid (자세히 볼 경우)
$ kubectl get pods -o yaml (yaml 포맷으로 볼 경우)
```

### multi-container pod 생성

##### yaml 예시

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multipod
spec:
  containers:
    - name: nginx-container
      image: nginx:1.14
      ports:
        - containerPort: 80
    - name: centos-container
      image: centos:7
      command:
        - sleep
        - "1000"
```

### pod container 접근

`kubectl exec multipod -c nginx-container -it -- /bin/bash`로 multipod의 nginx 컨테이너로 접근
<br/>

만약 `kubectl exec multipod -c centos-container -it -- /bin/bash` 접근 했을 경우 `curl localhost` 명령어를 실행 했을 경우 nginx-contrainer의 80포트의 내용을 응답한다.
**multipod에 두개의 컨테이너는 모두 같은 IP를 사용한다. (1개 Pod = 1개 IP)**

### pod log 확인

`kubectl logs multipod -c nginx-container`의 명령어로 원하는 컨테이너의 로그를 확인 할 수 있다.

## self healing (liveness probe)

- 예제 yaml (nginx의 80포트의 /(root path)로 주기적으로 상태 체크를 한다.)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: [pod 이름]
spec:
  containers:
    - name: nginx-container (container 이름)
      image: nginx:1.14
      livenssProbe:  ------------ (livenessProbe)
        httpGet:
          path: /
          prot: 80
```

### livenessProbe 매커니즘

1. httpGet: 지정한 ip주소, port, path에 http get 요청을 보내 해당 컨테이너가 응답하는지 확인. (반환 코드가 200아닐 경우 컨테이너를 다시 시작)
2. tcpSocket: 지정된 포트에 TCP연결을 시도. 연결 실패시 컨테이너 다시 시작.
3. exec: exec 명령을 전달하고 명령의 종료코드가 0이 아니면 컨테이너 다시 시작.

### livenessProbe 매개 변수

- periodSeconds: health check 반복 실행 시간 (초)
- initialDelaySeconds: Pod 실행 후 delay할 시간(초)
- timeoutSeconds: health check 후 응답 대기 시간(초)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: [pod 이름]
spec:
  containers:
    - name: nginx-container (container 이름)
      image: nginx:1.14
      ports:
      - containerPort: 80
        protocol: TCP
      livenssProbe:  ------------ (livenessProbe)
        httpGet:
          path: /
          prot: 80
        initialDelaySeconds: 15
        periodSeconds: 20
        timeoutSeconds: 1
        successThreshold: 1 -- 성공횟수
        failureThreshold: 3 -- 실패횟수
```

### init container를 적용한 pod

환경 구성에 대한 점검, db실행 등 실행 여부등을 확인하도록 구성하는 컨테이너<br/>

- 메인 앱 컨테이너 실행 전에 미리 동작시킬 컨테이너
- 메인 컨테이너가 실행되기 전에 사전 작업이 필요할 경우 사용
- init container가 run을 성공할 경우 연관된 main container가 실행된다.

##### init container 예시

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app.kubernetes.io/name: MyApp
spec:
  containers:
    - name: myapp-container
      image: busybox:1.28
      command: ["sh", "-c", "echo The app is running! && sleep 3600"]
  initContainers:
    - name: init-myservice
      image: busybox:1.28
      command:
        [
          "sh",
          "-c",
          "until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done",
        ]
    - name: init-mydb
      image: busybox:1.28
      command:
        [
          "sh",
          "-c",
          "until nslookup mydb.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for mydb; sleep 2; done",
        ]
```

위 예시에서 `init-myservice`가 실행이되면, `init-mydb` 가 실행되고 이후에 메인 컨테이너인 `myapp-container`가 실행된다.

### infra container 란?

pod 생성시 pause라는 컨테이너가 생성이 된다.

### static container 란?

api 서버 없이 특정 노드에 있는 kubelet 데몬에 의해 직접 관리되며 /etc/kubernetes/manifests/ 디렉토리에 k8s yaml파일을 저장 시 적용된다. (디렉토리구성은 달라질 수 있다.)<br/>

yaml 파일 삭제 시 pod도 삭제

- /var/lib/kubelet/config.yaml에서 staticpod의 디렉토리 경로를 볼 수 있다.

### pod에 리소스(cpu, memory) 할당하기

보안 및 ddos공격에 따른 각 pod에 node의 리소스를 할당하고 다른 pod가 동작될 수 있도록 할 수 있다. 바람직한 스케쥴링관리를 통해 node를 관리할 수 있다.

- Resource Requests : 파드를 실행하기 위한 최소 리소스양을 스케줄러에 요청. 스케줄러가 그에 맞는 node에 배정
- Resource Limits : 파드가 사용할 수 있는 최대 리소스 양을 제한, memeory limit을 초과해서 사용되는 파드는 종료되며 다시 스케줄링 된다.

##### 리소스 예시 yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: [pod 이름]
spec:
  containers:
    - name: nginx-container (container 이름)
      image: nginx:1.14
      ports:
        - containerPort: 80
          protocol: TCP
      resources:
        requests:
          cpu: 200m (1000m 가 1코어)
          memory: 250Mi (Ki, Mi, Gi 로 단위)
        limits:
          cpu: 1
          memory: 500Mi
```

### Pod안에 컨테이너 환경변수 설정하기

컨테이너에서 사용하는 환경변수의 값을 재설정 할 수 있다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: [pod 이름]
spec:
  containers:
    - name: nginx-container (container 이름)
      image: nginx:1.14
      ports:
        - containerPort: 80
          protocol: TCP
      env:
        - name: MYVAR
          value: "testvale"
```

변수의 name과 설정하고 싶은 값을 넣을 수 있다. 새로운 환경변수 및 기존의 환경변수 모두 설정할 수 있다.

### pod 구성 패턴의 종류

- pod를 구성하고 실행하는 패턴
- multi container pod

#### sidecar

혼자서는 구성할 수 없고 다른 컨테이너와 같이 구성해서 실행

#### adapter

외부 서비스의 데이터를 adapter 컨테이너가 받아서 같은 pod의 다른 컨테이너에게 전달하는 형식

#### ambassador

ambassador가 로드밸런스 및 라우터같은 역할을하여 pod내부의 컨테이너가 요청을 받고 ambassador가 다른 서비스로 요청하여 처리
