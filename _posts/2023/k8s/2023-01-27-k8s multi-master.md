---
layout: post
title: "kubernetes 기초 3 (multi-master)"
date: 2023-01-27
categories: [k8s]
---

# Multi-master(HA) 설치

HA 클러스터 운영으로 구성 다수의 중첩된 control plane을 구성.

### 구성 flow

master 노드는 load balancer를 통해 접근하고 각 master 노드의 etcd는 전체 master 노드들과 동기화를 통해 쿠버네티스의 정보의 sink를 맞춘다.<br/>
API는 loadbalancer를 통해 worker node에 노출

### 구성 순서

1. 모든 시스템에 CRI(docker) 설치
2. control plane, workder node - kubeadm 설치

   > - 설치 전 환경설정
   > - kubeadm, kubectl, kubelet 설치

3. LB(Load Balancer) 구성
4. kubeadm을 이용한 HA 클러스터 구성
   > - master1: kubeadm init 명령으로 초기화 - LB 등록
   > - master2, master3을 master1에 join
   > - CNI(Container Network Interface) 설치
   > - worker node를 LB통해 master와 join
   > - 설치된 시스템 확인

### LB 구성

테스트로 nginx를 사용하며 master의 단일 진입점으로 사용. LB서버에서 nginx를 설치한다.<br/>
config 파일 구성

```
mkdir /etc/nginx

cat << END > /etc/nginx/nginx.conf
events{}
stream{
  upstream stream_backend{
    least_com;
    server xx.xxx.xx.xx.xx:port;
    server xx.xxx.xx.xx.xx:port;
    server xx.xxx.xx.xx.xx:port;
  }

  server{
    listen {port};
    proxy_pass stream_backend;
    proxy_timeout 300s;
    proxy_connect_timeout 1s;
  }
}

END
```

도커로 nginx를 실행하고 LB 연결
`docker run --name proxy -v /etc/nginx/nginx.conf:/etc/nginx/nginx.conf:ro --restart=always -p 6443:6443 -d nginx`

### HA 클러스터 구성

master1: kubeadm init 명령으로 초기화 - LB 등록<br/>
`kubeadm init --control-plane-endpoint"{lb ip 및 port}" --upload-certs`<br/>
위 명령어 실행시 control-plane을 join해주는 명령어와 worker node를 join하는 명령어 2개를 알려준다.<br/>
(이전 싱글 클러스터에서는 worker node join명령어만 나오는것과 차이점)

join명령어로 master2, master3에서 실행하고 CNI를 master1에서만 설치하여 연결해준다.(CNI 플러그인은 공홍에서 참고하여 설치하며 쿠버로 deamon으로 전부 설치가 된다.)

worker node join명령어를 각 노드 서버에서 실행하여 연결해준다.
