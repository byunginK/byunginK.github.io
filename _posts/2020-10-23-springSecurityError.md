---
layout: post
title:  "Web Spring boot 정적 자원 경로"
date:   2020-10-23
categories: [web]
---

# MIME error 
##  static resource load

###### Spring Boot 에서 security setting을 한 뒤 static resource(ex : css, js, img ... )를 불러오는 부분에 있어서 MIME 에러가 발생하여 404ERROR가 발생하였다.

###### application.yalm 또는 application.properties에서 아래와 같이 static의 정적 resource를 설정 해 주어야 한다.

```yaml
mvc:
	static-path-pattern: static/**
resources:
	static-locations:
  - classpath:/static/
	add-mappings: true
```

###### 위 코드는 .yaml 방식이고 .properties라면 그에 맞는 코드를 작성해주면 된다.

##### 코드를 집어넣어도 404ERROR또는 MIME error가 계속 발생할 수 있는데, src/resources directory 안에 static이라는 이름의 폴더에 정적요소들을 위치시켰는지 다시 확인해보자.
