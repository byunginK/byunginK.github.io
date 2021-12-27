---
layout: post
title:  "Docker usage"
date:   2021-12-27
categories: [web]
---
# 도커 사용

## 이미지 찾기
### * 도커 레지스트리
도커 레지스트리에는 사용자가 사용할 수 있도록 데이터베이스를 통해 Image를 제공해주고 있음
누구나 이미지를 만들어 푸시할 수 있으며 푸시된 이미지는 다른 사람들에게 공유 가능

### * 도커 명령어로 검색

레지스트리에서 tomcat을 검색한다.

```bash
sudo docker search tomcat
```

### * 도커 이미지 다운로드

```bash
sudo docker pull tomcat
```

### * 로컬시스템에 있는 도커 이미지 확인
다운로드 받았던 도커의 이미지를 모두 확인 할 수 있다.

```bash
sudo docker images
```

## 도커 라이프사이클 도식화
![image](https://user-images.githubusercontent.com/65350890/147425783-e430a6b6-0fdc-4cd8-973c-8b2d7ee37aed.png)

아래 tomcat을 예시로 라이프사이클 진행

### * 도커 이미지 다운로드와 삭제

```bash
sudo docker pull consol/tomcat-7.0
sudo docker rmi consol/tomcat-7.0 
```

### * 톰캣 컨테이너 생성

```bash
# run은 create 와 start를 동시에 진행
sudo docker run -d --name tc tomcat # 톰캣 생성 및 실행
```

### * 실행중인 컨테이너 확인
```bash
sudo docker ps # 톰캣 컨테이너 확인
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
f6e513b399a6        tomcat              "catalina.sh run"   27 seconds ago      Up 26 seconds       8080/tcp            tc
```

### * 모든 컨테이너 확인
```bash
sudo docker ps -a # 모든 컨테이너 확인
```
컨테이너 실행, 중지, 삭제 시 컨테이너 선택은 container ID 또는 name으로 입력

### * 컨테이너 중지
```bash
sudo docker stop f6e513b399a6 # 컨테이너 실행 중지 
```

### * 컨테이너 삭제
```bash
sudo docker rm f6e513b399a6 # 컨테이너 삭제
```

## 레이어 개념
도커 이미지를 다운로드받아 run을 하게되면 기존의 이미지 레이어가 생성이 되고 그 위에 container layer가 생성이 된다. 

사용자가 컨테이너 안에서 읽고 쓰는 작업들은 모두 container layer에 적용이 되고 image layer는 변경 되지 않는다.

![image](https://user-images.githubusercontent.com/65350890/147429232-cce6fa6c-3299-4143-8cab-b935b40d126d.png)

아래 그림에서 이미지 A를 지운다 하더라도 이미지 B에서 레이어 A, B, C를 사용하고 있기 때문에 지워지지 않는다.
![image](https://user-images.githubusercontent.com/65350890/147429228-19c9c1f1-9b8e-4ad6-bc34-be3f6adcef13.png)

또한, 다른 사용자가 같은 이미지를 이용해서 container를 실행할 경우 다른 image layer만 생성되고, 다른 사용자가 생성한 container layer는 복사 되지 않는다.

(만약 직접 작상한 container layer를 같이 생성하고 싶다면 docker commit을 사용하여 새로운 이미지를 생성하여 사용 할 수 있다.)

### * 도커 이미지 정보 확인
nginx를 다운로드하여 확인해보는 예시

```bash
sudo docker pull nginx
sudo docker inspect nginx
```
##  도커 자주 쓰는 명령어
기본 포트라던지 shell 실행 명령은 inspect를 통해 확인 할 수 있다.

### * 포트포워딩
```bash
# -p 뒤에 포트를 설정 (80은 외부 포트 , 8080은 컨테이너 내부 포트)
sudo docker run -d --name tc -p 80:8080 tomcat
```

### * 컨테이너 내부 shell 실행
```bash
# -i : -t와 같이 사용해야 합니다. 표준입력을 활성화시키며 컨테이너와 연결되어있지 않더라도 표준입력을 유지합니다.
# -t : -i옵션과 같이 사용해야 합니다. TTY모드로 사용하며 bash를 사용하기 위해서는 꼭 필요합니다.
# tc는 컨테이너 names
sudo docker exec -it tc /bin/bash
```

### * 컨테이너 로그 확인
```bash
sudo docker logs tc
```

### * 호스트 및 컨테이너 간 파일 복사
```bash
sudo docker cp <path> <to container>:<path>
sudo docker cp <from container>:<path> <path>
sudo docker cp <from container>:<path> <to container>:<path>
```

### * 도커 컨테이너 모두 삭제

```bash
sudo docker stop `sudo docker ps -a -q`
sudo docker rm `sudo docker ps -a -q`
```

### * 임시 컨테이너 생성

```bash
sudo docker run -d -p 80:8080 --rm --name tc tomcat 
```

## 환경변수 사용
mysql을 예시로 환경변수를 사용하기

```bash
$ docker run --name some-mysql -e MYSQL_ROOT_PASSWORD='!qhdkscjfwj@' -d mysql
$ docker exec -it some-mysql mysql
password: !qhdkscjfwj@
mysql>
```

## 볼륨 마운트
볼륨 마운트 옵션 사용하여 로컬 파일 공유
```bash
docker run -v <호스트 경로>:<컨테이너 내 경로>:<권한>
```
권한 종류
- ro : 읽기 전용
- rw : 읽기 및 쓰기


### * nginx로 볼륨 마운트
```bash
sudo docker run -d -p 80:80 --rm -v /var/www:/usr/share/nginx/html:ro nginx
```