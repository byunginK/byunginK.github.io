---
layout: post
title:  "Web Spring boot 파일 업로드 경로"
date:   2020-10-26
categories: [web]
---

# Spring Boot File Upload

###### spring boot 를 사용할 경우 static파일을 통해 이미지, 파일, css , js등을 해당경로에 두고 사용 할 수 있다. 일반적으로 spring boot 는 패키징(jar, war)시 webapp 폴더를 만들어 사용하는것을 지양하고 있다. (경로를 읽지 않는다고 document에 명시 아래 참조)(단 war로 패키징할 경우 가능은하다)

![image3](https://user-images.githubusercontent.com/65350890/97144896-a24f9080-17a8-11eb-8266-d0f4e1581b18.png)

###### 따라서 static폴더에 경로를 지정하여 사용하는것이 좋고 multipart로 업로드시 파일 저장 경로를 잡아주어야 한다. 그러나 설정없이 저장을 하게 되면 임시파일이 같이 생성되고 저장경로에 실제 업로드한 파일 이외의 데이터가 같이 저장이 된다.(아래 예시처럼)

![image](https://user-images.githubusercontent.com/65350890/97144999-d0cd6b80-17a8-11eb-91ca-fdaf17e7092a.png)

#### 그래서 application.yaml 또는 application.properties에서 multipart(servlet)에 대한 초기 임시파일 저장 경로를 잡아주어야 한다.

###### 예 (application.yaml)
![image](https://user-images.githubusercontent.com/65350890/97145062-f0fd2a80-17a8-11eb-8df6-e74d81e1cbe1.png)

###### loaction 부분에 임시파일이 저장이 되고 업로드한 파일은 new file(지정경로)에 업로드한 파일만 저장된 것을 확인 할 수 있다.
