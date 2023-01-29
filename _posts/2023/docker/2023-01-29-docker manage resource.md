---
layout: post
title: "Docker 컨테이너 관리"
date: 2023-01-29
categories: [docker]
---

# 컨테이너 하드웨어 리소스 관리

기본으로 컨테이너는 호스트 하드웨어 리소스의 사용제한을 받지 않음.<br/>
컨테이너가 필요한 만큼만 할당해서 사용해야한다.

### docker command를 통해 제한할 수 있는 리소스

- CPU
- Memory
- Disk i/o

## Memory 리소스 제한

제한 단위는 b, k, m, g로 할당

| 옵션                 | 의미                                                                                                                                                                       |
| -------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| --memory, -m         | 컨테이너가 사용할 최대 메모리 양을 지정 `docker run -d -m 512m nginx:1.14`                                                                                                 |
| --memory-swap        | 컨테이너가 사용할 스왑 멤모리 영역에 대한 설정. 메모리 + 스왑. 생략시 메모리의 2개 설정 `docker run -d -m 200m --memory-swap 300m nginx:1.14` 실제 스왑 100m사용 이미 포함 |
| --memory-reservation | --memory 값보다 적은 값으로 구성하는 소프트 제한 값 설정 `docker run -d -m 1g --memory-reservation 500m nginx:1.14` 500메가 보장 최대 1기가 사용                           |
| --oom-kill-disable   | OOM killer가 프로세스 kill 하지 못하도록 보호                                                                                                                              |

- 리눅스 커널에서 물리적 메모리 이상을 사용하게 되면 out of memory killer가 작동하여 프로세스를 킬한다.

## CPU 리소스 제한

| 옵션          | 의미                                                                                                                                          |
| ------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| --cpus        | 컨테이너에 할당할 cpu core수를 지정 `docker run -d --cpus=".5" ubuntu1.14`                                                                    |
| --cpuset-cpus | 컨테이거 사용할 cpu나 코어를 지정 할당. cpu index는 0부터시작                                                                                 |
| --cpu-share   | 컨테이너가 사용하는 cpu비중을 기본 설정(1024)와 다르게 설정 `docker run -d --cpu-share 2048 ubuntu:1.14` ubuntu:1.14만 2048이고 나머지는 1024 |

## block i/o 제한

| 옵션                                    | 의미                                                  |
| --------------------------------------- | ----------------------------------------------------- |
| --blkio-weight, --blkio-weight-device   | block io의 quota를 설정 100~1000까지 선택 default:500 |
| --device-read-bps, --device-write-bps   | 특정 디바이스에 대한 읽기와 쓰기작업의 초당 제한설정  |
| --device-read-iops, --device-write-iops | read/write 속도의 쿼터를 설정한다.                    |

# 리소스 확인하는 모니터링

### docker monitoring commands

- docker stat: 실행중인 컨테이너의 런타임 통계를 확인
- docker event: 토커 호스트의 실시간 event 정보를 수집해서 출력

### cAdvisor

- 구글에서 만든 모니터링 툴
