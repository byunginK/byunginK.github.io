---
layout: post
title: "spring boot 무중단 배포"
date: 2023-08-05
categories: [spring]
---

### 1. 무중단 배포

\- 서비스를 중지하지 않고, 배포를 계속하는 것을 **무중단 배포**라고 합니다.

-   무중단 배포 방식들
    1.  **[AWS의 Blue-Green 무중단 배포](https://sangwook.github.io/2014/01/28/zero-downtime-blue-green-deployments-aws.html)**
    2.  **[도커를 이용한 무중단 배포](https://subicura.com/2016/06/07/zero-downtime-docker-deployment.html)**
    3.  **L4 스위치를 이용한 무중단 배포**
    4.  **Nginx를 이용한 무중단 배포**

> 해당 글에서는 Nginx를 사용하여 무중단 배포를 구축하겠지만, 추후 Docker(도커)도 사용해보고 포스팅할 수 있도록 하겠습니다.

### 2. Nginx를 이용한 무중단 배포

\- 위의 방식 중 가장 저렴하다.

\- 배포를 위한 AWS EC2 인스턴스가 한 개 더 필요하지 않다.

\- 클라우드 인프라가 구축되어 있지 않아도 쓸 수 있다.

#### 2-1. 구조설명

-   사용하고 있는 EC2 서버에 Nginx 1대와 스프링부트 jar 2대를 사용하겠습니다.
-   Nginx에는 80(http), 443(https) 포트를 할당합니다.
-   스프링부트 jar1에는 8081포트로 , 스프링부트 jar2에는 8082포트로 실행합니다.(포트는 원하시는 포트를 사용하시면 됩니다.)
-   구조는 아래의 그림같이 형성됩니다.

![](https://blog.kakaocdn.net/dn/bL6cSi/btq3t3TslJs/7jkbcw5BQgzHKC7wRl8PFK/img.png)


-   위 그림의 동작 과정은 다음과 같습니다.
    1.  사용자는 80, 443 포트로 서비스에 접속합니다.
    2.  Nginx는 해당 요청을 받아 현재 동작중인 스프링부트 Jar1(Port: 8081)로 전달합니다.
    3.  스프링부트 Jar2(Port: 8082)는 현재 동작중이지 않기 때문에 받지 못합니다.

![](https://blog.kakaocdn.net/dn/bByVk0/btq3osUeU6U/GufUg993KbBvGORkX8DBFK/img.png)


-   위 그림의 신규 버전 배포시 동작과정은 다음과 같습니다.
    1.  2.0 버전으로 신규 배포가 진행되면 현재 동작중이지 않은 스프링부트 Jar2(8082)로 배포합니다.
    2.  배포하는 동안에는 사용자는 스프링부터 Jar1(8081)를 계속해서 바라보고 있는 중입니다.
    3.  배포가 정상적으로 끝난다면, 스프링부트 Jar2(8082)의 구동 여부를 확인합니다.
    4.  정상 구동 중이라면 nginx는 스프링부트 Jar2(8082)를 바라보도록 설정합니다.

-   배포에 문제가 생길 시에는 정상 구동 중인 스프링부트 Jar로 돌아갑니다.

![](https://blog.kakaocdn.net/dn/Apjd8/btq3suKxQHW/0jXj3k5n5qDsilU9OgfQlK/img.png)


-   위의 그림 동작 과정은 다음과 같습니다.
    1.  2.1 버전으로 신규 배포가 진행되면 현재 동작중이지 않은 스프링부트 Jar1(8081)로 배포합니다.
    2.  배포하는 동안에는 사용자는 스프링부터 Jar2(8082)를 계속해서 바라보고 있는 중입니다.
    3.  배포 도중 문제가 생겼다면, nginx는 그대로 스프링부트 Jar2(8082)를 바라보도록 합니다.
    4.  배포의 문제를 확인하고 다시 배포를 진행합니다.

-   그럼 이제 무중단 배포까지 구축이 된다면 전체적인 서비스의 구조는 아래 그림과 같아집니다.

![](https://blog.kakaocdn.net/dn/bVIHeJ/btq3mwcfIhB/5CcrEjqmXPpoB2ca8iYT5k/img.png)


### 3. EC2에 Nginx 설치

\- ssh 접속을 통해 사용하는 EC2에 접속합니다.

\- 다음의 명령어를 통해 Nginx를 설치합니다.

> sudo yum install nginx

\- 만약, 아래의 이미지와 같이 **nginx package가 없다**는 메시지를 받거나, 실패한다면 **3-1를 진행**하여 설치합니다.

  (정상적으로 설치가 진행되셨다면, 3-1는 패스해주세요.)

![](https://blog.kakaocdn.net/dn/6i0dK/btq3qgeW33T/iA6MK1BOiWfiOvzxr5Kxu0/img.png)

#### 3-1. Nginx 설치
-   Nginx를 설치하기 위한 Repository를 추가하겠습니다.
-   아래의 명령어를 통해 nginx.repo 파일을 생성합니다.

> $ sudo vi /etc/yum.repos.d/nginx.repo

-   아래의 내용을 입력 후 저장합니다.

```
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/7/$basearch/
gpgcheck=0
enabled=1
```

-   추가한 Repository를 확인합니다.

> $ yum info nginx

![](https://blog.kakaocdn.net/dn/d4tPh5/btq3q7B3Sn7/SO72rq8Vrz11IeBUQKkfWK/img.png)

-   정상적으로 확인 된다면, nginx 를  설치합니다.

> sudo yum install nginx

\- 설치가 완료되었다면, Nginx 를 실행합니다.

> sudo serivce nginx start

\- 정상적으로 Nginx가 실행되었는지 확인합니다.

> ps -ef | grep nginx

![](https://blog.kakaocdn.net/dn/bOh6rk/btq3mp5vOmY/7Xfkel5VtS5RJfu4G1SBE0/img.png)

\- Nginx 가 실행되고 있다면, EC2(퍼블릭 DNS)로 접속하여 Nginx가 노출되고 있는지 확인해보도록 하겠습니다.

   (EC2의 퍼블릭DNS는 자주 확인하게 되므로 즐겨찾기하여 쉽게 이동할 수 있도록 합니다) 

![](https://blog.kakaocdn.net/dn/vhQty/btq3mNZoAle/cxpbKkVLx1GaCMb5Gg9Kl1/img.png)

\- Nginx가 정상적으로 노출되고 있지만, 해당 페이지는 Nginx의 Default 페이지므로, Nginx가 저희의 스프링부트 프로젝트를 바라보도록 리버시 프록시 설정을 하도록 하겠습니다.

\- nginx 설정파일을 열도록 하겠습니다.

> sudo vi /etc/nginx/nginx.conf

\- nginx.conf 파일에서 **server 아래에 location /** 부분을 찾아서 다음의 내용을 추가합니다.

![](https://blog.kakaocdn.net/dn/mohsE/btq3n9NUzT0/8C7iBANvZ6ohDZkqtHkf1k/img.png)

```
proxy_pass http://localhost:8080;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header Host $http_host;
```

-   proxy\_pass : Nginx에 요청이 오면 Nginx는 해당 요청을 http://localhost:8080로 전달
-   proxy\_set\_header XXX : 실제 요청 데이터를 header의 각 항목에 할당

\- 만약, nginx.conf 파일에 아래 이미지같이 **server 아래에 location /** 부분이 없다면, 다음의 파일을 열어 진행하시면 됩니다.

![](https://blog.kakaocdn.net/dn/0WYPb/btq3mZec5Dv/Er0mxG48acH3LJ0W9QilWK/img.png)

> sudo vi /etc/nginx/conf.d/\*.conf  
> (\*.conf  --> \* 의미는 전체를 의미합니다. 만약 conf 파일이 여러개라면 default.conf 파일을 확인하시면 됩니다.)

\- conf 파일 수정이 완료되었다면, Nginx를 재시작합니다.

> sudo service nginx restart

\- 다시 EC2(퍼블릭 DNS)로 접속하여 페이지를 확인합니다.

![](https://blog.kakaocdn.net/dn/bGw2Al/btq3mxvwUEm/OL486JSFHoo43NNW2mj3AK/img.png)

\- Nginx의 Default 페이지가 아닌 배포한 스프링부트 프로젝트로 접속되는 것을 확인합니다.

### 4. 프로젝트 set1, set2 Profile 설정하기

\- 실제 서비스에서는 로컬, 개발, 운영 서버 등으로 환경들이 분리되어 있고, 각각 사용하는 DB, API 주소 등이 다릅니다.

\- properties, yml 파일을 통해 환경들을 분리하여 사용하도록 하겠습니다.

\- 실행중인 프로젝트의 Profile이 뭔지 확인할 수 있는 API를 만들도록 하겠습니다.

\- WebRestController.java란 파일을 생성하여 아래의 API 메소드를 추가하도록 하겠습니다.

```
import org.springframework.core.env.Environment;

@RestController
@AllArgsConstructor
public class WebRestController {

    private Environment env;

    @GetMapping("/profile")
    public String getProfile(){
        return Arrays.stream(env.getActiveProfiles())
                .findFirst()
                .orElse("");
    }
}
```

\- 프로젝트의 환경설정 값을 다루는 **Environment** **Bean을 DI 받아 현재 활성화된 Profile을 반환**합니다.

\- Profile을 확인하기 위해 운영환경의 yml 파일을 생성하여 확인해보도록 하겠습니다.

\- 운영 환경의 yml 파일은 프로젝트 내부가 아닌 **외부인 로컬PC 디렉토리**에 생성하겠습니다.

> **참고!!!**  
> 운영환경 설정파일은 절대 **프로젝트 내부에 포함시키지 않습니다**.  
> 혹여, 운영환경 설정파일을 Git Push를 한번이라도 하셨다면 프로젝트를 삭제하시는고 새로 생성하도록 추천드립니다.  
> Git은 한번이라도 커밋 되면 커밋 이력이 남기 때문에 단순히 파일 삭제만 한다고 내용이 사라지지 않습니다.  
> Github 같이 오픈된 공간에 운영환경의 설정 (Database 접속정보, 세션저장소 접속정보, 암호화 키 등등)들이 있다면, 새로 Repository를  
> 생성하여, gitIgonre에 관련 파일을 추가하고 push 하여 사용하도록 합니다.  
> 만약, AWS 접속 정보같은 것이 올라간다면 악의적으로 접속하여 요금이 부과되는 일이 발생할수도 있습니다.

\- 원하시는 디렉토리에 real-application.yml 파일을 생성하도록 하겠습니다.

\- 해당 글에서는 /app/config/springboot-webservice/ 위치에 생성하였습니다.

> sudo mkdir /app  
> sudo mkdir /app/config  
> sudo mkdir /app/config/springboot-webservice
> 
> sudo vi /app/config/springboot-webservice/real-application.yml

\- real-application.yml 에는 아래의 코드를 입력하여 저장합니다.

```
---
spring:
  profiles: set1
server:
  port: 8081

---
spring:
  profiles: set2

server:
  port: 8082
```

\- **set1 Profile은 8081포트를 set2 profile은 8082 포트**를 갖도록 설정하였습니다.

\- 다시 프로젝트에서 스프링부트 실행파일인 Application.java 에서 생성한 yml 파일을 호출하도록 수정합니다.

   (기본적으로 스프링부트 실행파일은 프로젝트이름Application.java 입니다.)

```
@EnableJpaAuditing // -> JPA Auditing 활성화
@SpringBootApplication
public class Application {

    public static final String APPLICATION_LOCATIONS = "spring.config.location="
            + "classpath:application.yml,"
            + "/app/config/springboot-webservice/real-application.yml";

    public static void main(String[] args) {
        new SpringApplicationBuilder(Application.class)
                .properties(APPLICATION_LOCATIONS)
                .run(args);
    }
}
```

\- 프로젝트가 실행될때, 프로젝트 내부에 있는 application.yml과 프로젝트 외부(로컬PC)에 위치한 /app/config/springboot-webservice/real-application.yml를 불러와 설정파일로 등록하여 실행하도록 하였습니다.

\- 정상적으로 동작하는지 확인해보도록 하겠습니다.

\- Intell J를 사용하신다면, **command + shift + a** 를 통해 **Edit** **Configuration** 검색하여 실행합니다.

  (Eclipse 에서는 확인해보지 못하였습니다...)

![](https://blog.kakaocdn.net/dn/AEVKA/btq3sucI2xT/GEWTmESFdVv0IKeQrmA9TK/img.png)

\- Application을 선택 후 좌측 상단의 Copy 버튼을 클릭해서 설정 내용을 복사합니다.

![](https://blog.kakaocdn.net/dn/cKLRrW/btq3mxWwSh0/0hmrT601oMc1wu3fa5eW91/img.png)

\- 복사된 설정 내용을 아래와 같이 set1을 Profile로 지정한 실행환경으로 수정합니다.

![](https://blog.kakaocdn.net/dn/wDAbf/btq3r2mZ9mr/kg9v0nzrrXqPQevmRA76Vk/img.png)

\- 해당 설정을 저장후, 해당 실행환경으로 프로젝트를 실행합니다.

![](https://blog.kakaocdn.net/dn/bPkmD9/btq3oSSHcu1/XE0HXXHtogwQKLk330FYj0/img.png)

![](https://blog.kakaocdn.net/dn/dZq0yd/btq3nJWgn5E/E4T9p1Gy4JCjPKJ167Xbkk/img.png)

\- 브라우저를 통해서도 set 1 이 정상적으로 노출되는지 확인해보도록 합니다.

![](https://blog.kakaocdn.net/dn/pOZ1l/btq3n9NVyPT/xAAkGVDIiCpxCAPfvtsGk1/img.png)

\- 정상적으로 set1 이 반환되는 것이 확인이 되었다면, EC2 에서도 똑같이 설정파일을 추가하도록 하겠습니다.

\- ssh로 EC2 에 접속하여, 로컬에서와 같은 경로로 real-application.yml 파일을 생성하여 설정값을 등록합니다.

> sudo mkdir /app  
> sudo mkdir /app/config  
> sudo mkdir /app/config/springboot-webservice
> 
> sudo vi /app/config/springboot-webservice/real-application.yml

\- 로컬에서는 포트 번호를 8081, 8082로 입력하였지만, EC2에서는 본인이 사용하고자 하는 포트번호를 입력합니다.(외부 공유X)

\- 파일을 생성하였다면, 로컬에서 개발한 내용을 푸시후에 정상적으로 Profile이 반환되는지 확인하도록 하겠습니다.

![](https://blog.kakaocdn.net/dn/vNZ6a/btq3udPk0IS/LkkJjctwcCCydBrP1vNiTK/img.png)

\- EC2는 현재 profile 옵션을 주지 않고 실행했기 때문에 기본값인 local이 적용됩니다.

### 5. 배포스크립트 작성하기

\- 무중단 배포와 관련된 파일을 관리할 디렉토리와 스크립트 파일을 생성하도록 하겠습니다.

\- 지금까지, 1번째 배포 디렉토리로 git, 2번째 배포 디렉토리로 travis 를 생성했습니다.

\- 3번째는 nonstop 이란 이름으로 디렉토리를 생성하도록 하겠습니다.

> sudo mkdir ~/app/nonstop

\- 배포 스크립트가 정상적으로 동작하는지 테스트 해보기 위해 기존에 받아둔 가장 최신의 스프링 프로젝트.jar를 복사합니다.

> mkdir ~/app/nonstop/springboot-webservice  
> mkdir ~/app/nonstop/springboot-webservice/build  
> mkdir ~/app/nonstop/springboot-webservice/build/libs
> 
> cp ~/app/travis/build/build/libs/\*.jar ~/app/nonstop/springboot-webservice/build/libs/

![](https://blog.kakaocdn.net/dn/bpM0mD/btq3oSrEDDD/gH35AVT5AqTtkBbT7ym0j0/img.png)

\- jar파일을 모아둘 디렉토리를 따로 생성합니다.

> mkdir ~/app/nonstop/jar

\- 배포 스크립트 파일을 생성하도록 하겠습니다.

> sudo vi ~/app/nonstop/deploy.sh

\- 스크립트 내용은 아래와 같습니다.

```
#!/bin/bash
BASE_PATH=/home/ec2-user/app/nonstop
BUILD_PATH=$(ls $BASE_PATH/springboot-webservice/build/libs/*.jar)
JAR_NAME=$(basename $BUILD_PATH)
echo "> build 파일명: $JAR_NAME"

echo "> build 파일 복사"
DEPLOY_PATH=$BASE_PATH/jar/
cp $BUILD_PATH $DEPLOY_PATH

echo "> 현재 구동중인 Set 확인"
CURRENT_PROFILE=$(curl -s http://localhost/profile)
echo "> $CURRENT_PROFILE"

# 쉬고 있는 set 찾기: set1이 사용중이면 set2가 쉬고 있고, 반대면 set1이 쉬고 있음
if [ $CURRENT_PROFILE == set1 ]
then
  IDLE_PROFILE=set2
  IDLE_PORT=8082
elif [ $CURRENT_PROFILE == set2 ]
then
  IDLE_PROFILE=set1
  IDLE_PORT=8081
else
  echo "> 일치하는 Profile이 없습니다. Profile: $CURRENT_PROFILE"
  echo "> set1을 할당합니다. IDLE_PROFILE: set1"
  IDLE_PROFILE=set1
  IDLE_PORT=8081
fi

echo "> application.jar 교체"
IDLE_APPLICATION=$IDLE_PROFILE-jaewon-study.jar
IDLE_APPLICATION_PATH=$DEPLOY_PATH$IDLE_APPLICATION

ln -Tfs $DEPLOY_PATH$JAR_NAME $IDLE_APPLICATION_PATH

echo "> $IDLE_PROFILE 에서 구동중인 애플리케이션 pid 확인"
IDLE_PID=$(pgrep -f $IDLE_APPLICATION)

if [ -z $IDLE_PID ]
then
  echo "> 현재 구동중인 애플리케이션이 없으므로 종료하지 않습니다."
else
  echo "> kill -15 $IDLE_PID"
  kill -15 $IDLE_PID
  sleep 5
fi

echo "> $IDLE_PROFILE 배포"
nohup java -jar -Dspring.profiles.active=$IDLE_PROFILE $IDLE_APPLICATION_PATH &

echo "> $IDLE_PROFILE 10초 후 Health check 시작"
echo "> curl -s http://localhost:$IDLE_PORT/actuator/health "
sleep 10

for retry_count in {1..10}
do
  response=$(curl -s http://localhost:$IDLE_PORT/actuator/health)
  up_count=$(echo $response | grep 'UP' | wc -l)

  if [ $up_count -ge 1 ]
  then # $up_count >= 1 ("UP" 문자열이 있는지 검증)
      echo "> Health check 성공"
      break
  else
      echo "> Health check의 응답을 알 수 없거나 혹은 status가 UP이 아닙니다."
      echo "> Health check: ${response}"
  fi

  if [ $retry_count -eq 10 ]
  then
    echo "> Health check 실패. "
    echo "> Nginx에 연결하지 않고 배포를 종료합니다."
    exit 1
  fi

  echo "> Health check 연결 실패. 재시도..."
  sleep 10
done
```

\- 코드의 내용은 다음과 같습니다.

![](https://blog.kakaocdn.net/dn/R8QGw/btq3mZyvJNW/YEaS76xgwYF7Q9YsSfDYe0/img.png)

![](https://blog.kakaocdn.net/dn/ni3ia/btq3n9mSAVY/3I2s5cvzYtXoIGLxKVwk11/img.png)

\- 스크립트를 저장 후 실행권한을 주어 실행해보도록 하겠습니다.

> sudo chmod +x ~/app/nonstop/deploy.sh
> 
> ./app/nonstop/deploy.sh

![](https://blog.kakaocdn.net/dn/76TFd/btq3q7hOcnZ/3upCKxmth8F1zTvrQZzuK0/img.png)

\- 스크립트는 정상적으로 동작하였습니다.

\- 프로젝트도 실행이 되었는지 확인해보도록 하겠습니다.

> ps -ef | grep java

![](https://blog.kakaocdn.net/dn/sY7Tw/btq3strod9D/eK30ZV121OAsIRYjSSxwB0/img.png)

### 6. Nginx 동적 프록시 설정하기

\- 아직까지는 배포가 진행되어도, set1 Profile 만 바라보고 있습니다.

\- Nginx가 set1과 set2를 번갈아가면서 바라보도록 하는(프록시) 설정을 만들어보도록 하겠습니다.

\- EC2에서 설치한 Nginx의 설정을 수정하도록 하겠습니다.

\- nginx의 디렉토리로 이동합니다.

> cd /etc/nginx

![](https://blog.kakaocdn.net/dn/bxmoM8/btq3t4ZaRqr/W2fPHDQlZIfnlW57V7F9w1/img.png)

\- /etc/nginx 디렉토리가 Nginx 설정에 관련된 디렉토리입니다.

\- Nginx가 동적으로 **Proxy Pass 를 변경**하도록 설정을 수정해보도록 하겠습니다.

> sudo vi /etc/nginx/nginx.conf  
> 또는  
> sudo vi /etc/nginx/conf.d/default.conf

\- 위에서 proxy 관련하여 추가한 **location /** 부분에 다음과 같이 수정합니다.

```
include /etc/nginx/conf.d/service-url.inc;

location / {
    proxy_pass $service_url;
            ...
            proxy_set_header X-Real-IP $remote_addr;
```

![](https://blog.kakaocdn.net/dn/BGcA4/btq3oS6gGDw/6S9JTpPLjjTJrfxKdBEFIK/img.png)

-   include /etc/nginx/conf.d/service-url.inc;
    1.  service-url.inc 파일을 include 시킵니다.(Java의 import 패키지)
    2.  이렇게 할 경우 nginx.conf에서 **service-url.inc에 있는 변수들을 그대로 사용**할 수 있습니다.
-   proxy\_pass $service\_url;
    1.  service-url.inc에 있는 service\_url 변수를 호출합니다.

\- service-url.inc 파일을 생성합니다.

> sudo vi /etc/nginx/conf.d/service-url.inc

\- 코드는 아래와 같습니다.

```
# 포트는 사용하고자 하는 set1 포트
set $service_url http://127.0.0.1:8081;
```

\- 저장 후 , 변경 내용이 반영되도록 nginx를 재시작합니다.

> sudo nginx service restart

\- 정상적으로 동작하는지 테스트 합니다.

> curl -s localhost/profile

![](https://blog.kakaocdn.net/dn/ojQUy/btq3mwi3Och/LVRD6CX9Inm9IfJakYEq10/img.png)

### 7. Nginx 스크립트 작성하기

\- 동적 프록시 환경이 구축된 Nginx을 **배포 시점에 바라보는 Profile을 자동으로 변경**하는 스위치(switch) 스크립트를 생성하겠습니다.

> sudo vi ~/app/nonstop/switch.sh

\- 스크립트 내용은 아래와 같습니다.

```
#!/bin/bash
echo "> 현재 구동중인 Port 확인"
CURRENT_PROFILE=$(curl -s http://localhost/profile)

# 쉬고 있는 set 찾기: set1이 사용중이면 set2가 쉬고 있고, 반대면 set1이 쉬고 있음
if [ $CURRENT_PROFILE == set1 ]
then
  IDLE_PORT=8082
elif [ $CURRENT_PROFILE == set2 ]
then
  IDLE_PORT=8081
else
  echo "> 일치하는 Profile이 없습니다. Profile: $CURRENT_PROFILE"
  echo "> 8081을 할당합니다."
  IDLE_PORT=8081
fi

echo "> 전환할 Port: $IDLE_PORT"
echo "> Port 전환"
echo "set \$service_url http://127.0.0.1:${IDLE_PORT};" |sudo tee /etc/nginx/conf.d/service-url.inc

PROXY_PORT=$(curl -s http://localhost/profile)
echo "> Nginx Current Proxy Port: $PROXY_PORT"

echo "> Nginx Reload"
sudo service nginx reload
```

\- 저장 후, 실행권한을 줍니다.

> sudo chmod +x ~/app/nonstop/switch.sh

\- 해당 스크립트를 실행하기전에 set1, set2를 두개다 실행하도록 하겠습니다.

   (현재는 set1만 실행된 상태이며, 해당 스크립트 실행시 set2도 실행될 것 입니다.)

> ~/app/nonstop/deploy.sh

![](https://blog.kakaocdn.net/dn/Ztp8n/btq3qMdF2Oe/oj6EMqCXv0wZaqQNqc0AqK/img.png)

\- 위 그림과 같이 프로젝트가 set1, set2가 구동중이며, Nginx는 현재 set1을 바라보고 있습니다.

\- swtich.sh 를 실행하여, profile 이 변하는지 확인하도록 하겠습니다.

> ~/app/nonstop/switch.sh

\- EC2 (퍼블릭DNS)로 접속(http://주소/profile) 하여, 정상적으로 Profile이 반환되는지 확인합니다.

\- 정상적으로 반환되는지 확인이 되면, deploy.sh와 switch.sh를 합쳐 deploy.sh가 실행되면 다음으로 switch.sh가 자동으로 실행되도록 수정하도록 하겠습니다.

> sudo vi ~/app/nonstop/deploy.sh

\- 기존 코드 맨 밑에 아래 코드를 추가합니다.

```
  echo "> Health check 연결 실패. 재시도..."
  sleep 10
done
...

# 아래 추가
echo "> 스위칭"
sleep 10
/home/ec2-user/app/nonstop/switch.sh
```

![](https://blog.kakaocdn.net/dn/biLbEV/btq3nbMuKr0/OxUk3kkXrvS3Phy9dZjSd1/img.png)

\- 수정 후 해당 파일을 실행하여 테스트 해보겠습니다.

> ~/app/nonstop/deploy.sh

![](https://blog.kakaocdn.net/dn/waw7M/btq3n9mTTcW/GTa8Nu8F7w8m9oqUI0saKK/img.png)

### 8. 실제 무중단 배포하기

\- 기존에 구동에 배포된 jar 파일들을 삭제합니다.

> sudo rm ~/app/nonstop/springboot-webservice/build/libs/\*.jar

\- 기존에 /travis 로 설정되어 있는 무중단 배포 설정으로 변경하도록 하겠습니다.

\- 프로젝트에서 이전에 생성한 execute-deploy.sh 파일에서 아래와 같이 수정하도록 하겠습니다.

```
#!/bin/bash
/home/ec2-user/app/nonstop/deploy.sh > /dev/null 2> /dev/null < /dev/null &
```

-   travis로 잡혀있던 deploy.sh 를 nonstop 디렉토리의 deploy.sh 으로 변경합니다.

\- 프로젝트에서 appspec.yml 도 아래와 같이 /nonstop/springboot-webservice/로 변경합니다.

```
version: 0.0
os: linux
files:
  - source:  /
    destination: /home/ec2-user/app/nonstop/springboot-webservice/
    overwrtie: yes
    
    ...
```

\- 변경 사항을 확인하기 위해, 버전을 또 수정하고 git commit & push 를 진행합니다.

![](https://blog.kakaocdn.net/dn/rjMGf/btq3udaLCsb/yP7ELvUlkgVZpPDLZ9TWUk/img.png)

\- travis ci 와 code deploy 를 통해 진행상황을 확인하고, 배포가 정상적으로 되었다면, 확인을 합니다.

![](https://blog.kakaocdn.net/dn/VP8H2/btq3osUjKbo/CckqKSv8lGeBWdMvcMxqR0/img.png)

\- 정상적으로 반영이 되었습니다.
