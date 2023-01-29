---
layout: post
title: "Docker 컨테이너"
date: 2023-01-29
categories: [docker]
---

# Dockerfile

컨터이너를 만들 수 있도록 명령어 집합. 명령어는 top-down해석. 대소문자를 구분하지 않으나, 가독성을 위해 대소문자를 구분해서 사용함.

### 문법

- \# : 주석
- FROM : base image
- COPY : 컨테이너 빌드시 호스트의 파일을 컨테이너로 복사
- ADD : 컨테이너 빌드시 호스트의 파일(tar, url 포함)을 컨테이너로 복사
- RUN : 컨테이너 빌드를 위해 base image에서 실해할 commands
- MAINTAINER : 이미지를 생성한 사람의 이름 및 정보
- LABEL : 컨테이너 이미지에 대한 정보
- WORKDIR : 컨테이너 빌드시 명령이 실행될 작업 디렉터리 설정
- ENV : 환경변수 지정
- USER : 명령 및 컨테이너 실행 시 적용할 유저 설정
- VOLUME : 파일 또는 디렉토리를 컨테이너의 디렉토리 마운트
- EXPOSE : 컨테이너 동작 시 외부에서 사용할 포트 지정
- CMD : 컨테이너 동작 시 자동으로 동작시키고 싶은 스크립트, 서비스 지정
- ENTRYPOINT : CMD와 함계 사용하면서 command 지정시 사용

## hub (컨테이너 레포지토리) push

- 컨테이너를 빌드
- `docker login`
- `docker push 이미지(컨테이너)` <br/>
  위와 같이 간단한 흐름으로 저장 가능하다.

## 컨테이너 볼륨

컨테이너가 만들어주는 데이터를 영구적 보존 <br/>
`docker run -d --name db -v /{local host 저장장소}:/{컨테이너 내부 저장장소} mysql:letest`

### 컨테이너 끼리 데이터 공유

1. `docker run -v /webdata:/webdata -d --name df {이미지}`
2. `docker run -d -v /webdata:/usr/share/nginx/html:ro -d ubuntu:latest` ro를 통해 read only 모드를 활성화 시켜 공유할 수 있다.

## 컨테이너 통신

docker는 컨테이너 ip 대역과 서버 host를 연결하는 virtual ethernet bridge가 있다. docker0 bridge는 컨테이너 게이트웨이 역할을 하게된다. _모든 컨테이너는 외부 통신을 docker0 통해 진행_ 한다.

### 포트 포워딩

컨테이너 포트를 외부로 노출 시켜 외부연결 허용<br/>
`docker run -p hostPort:containerPort`로 포트 노출

### 사용자 정의 네트워크 생성

`docker network create --driver bridge --subnet 192.168.100.0/24 --gateway 192.168.100.1 mynet`으로 서브넷과 게이트웨이 ip도 설정할 수 있다. 만약 옵션으로 설정하지 않을 경우 기본 docker0에서 순차적으로 증가된 값으로 부여<br/>
`docker run -d --name appjs --net mynet --ip 192.168.100.100 -p 8080:8080 {컨테이너}` --net으로 사용자 네트워크를 추가하고 --ip의 옵션으로 고정 ip를 사용할 수도 있다.

### 컨테이너끼리 통신

예를 들어 app 서비스와 db의 컨테이너끼리 통신을 하기위해서는 --link 옵션을 사용한다. `docker run -d --name wordpress --link containerName:aliasName ... `으로 연결

## docker compose

여러 컨테이너를 일괄적으로 정의하고 실행할 수 있는 툴 docker-compose.yaml의 파일을 읽어서 컨테이너 관리

```
docker-compose up -d --- background 모드로 yaml 실행
docker-compose ps
docker-compose scale 컨테이너명=개수
docker-compose down ---- 컨테이너 rm과 같은 의미
```
