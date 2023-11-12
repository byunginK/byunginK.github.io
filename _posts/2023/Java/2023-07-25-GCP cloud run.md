---
layout: post
title: "GCP Cloud run에 대해"
date: 2023-07-25
categories: [GCP]
---
# GCP deep dive

- compute engine
- k8s engine
    - node는 compute engine을 사용하여 구성
- cloud run
- cloud function
    - code base, (aws lamda와 대칭) / 내부적인 환경은 cloud run으로 구성

# Cloud run

### 1. 정의

- no infra management
- auto-scailng
- speed to Market (빠르게 배포 가능)

### 2. 용어

- Service: region내 여러 zone으로 자동 복제, 고유한 endpoint 유지, 자동확장
- Revisions: 배포시 마다 revision(컨테이너 이미지), 환경변수, 동시성(concurrency) 설정 변경 시 신규 생성
- Instance: 요청받은 Revision은 인스턴스를 생성 및 자동 확장/축소, Concurrency에 따라 한 instance가 여러 요청 처리

### 3. 주의사항

- 서비스 인스턴스는 REST API 로 보통 사용 (하나의 특정 포트를 리스너)(side car 컨테이너 기능 추후 예정) 사이드카 ingress도 포트하나 지정으로 리슨
- timeout 전에 요청을 처리하고 응답해야한다.
    - 응답 이후의 background 작업은 보장이 안됨
- stateless 어플리케이션 사용 권장

### 4. 두가지 기능

1. services
    
    request base application, 기본적으로 tls 적용
    
2. Jobs
    
    batch형식의 타입의 기능. 스케쥴링도 가능
    

### 5. rollout/ rollback

canali 배포가 가능 (빠른 롤백 가능)

트래픽에 대한 가중치를 설정 가능, 다양한 배포 전략 가능, tag를 활용한 blue/green 배포가능 cli로 tag를 붙여 실제 트래픽 설정 이전에 start 테스트 가능

### 6. start

- cold start로 인해 cpu booster를 사용하더라도 1코어에서는 효과가 없을 수 있어 처음부터 높은 코어를 가지고 가는 전략도 사용할 수 있다.

(초반 cloud run의 코어 수를 4개를 설정하여 booster를 사용하면 8개 (최대). cold start를 최소화 가능 , 또는 min instance를 1로 잡고 always allocated cpu로 설정하거나)

### 7. 모니터링

cloud mornitoring 의 metrix을 커스텀해서 볼 수 았다.

always allocated cpu의 옵션을 주었을때 Idle상태가 없이 진행. 

always를 해놓고 min instance 1로 하면 shut down없이 계속. 그런게 없다면 15분 뒤에 shut down

### 8. health check

k8s의 probe기능을 활용한 cloud run에서도 api 체크 가능