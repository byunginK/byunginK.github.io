---
layout: post
title: "Spring MVC 요청 흐름"
date: 2023-01-30
categories: [JAVA, Spring]
---

# Spring MVC Http 요청 흐름

### Http 요청에 따른 Spring 처리 흐름

![image](https://user-images.githubusercontent.com/65350890/215465604-b7ebb775-f473-41d5-aa36-6495e076d6aa.png)

클라이언트가 HTTP 요청을 보내면, WAS는 소켓을 통해 클라이언트와 TCP/IP연결한다.
이후 WAS에서는 그림에서와 같은 과정이 발생한다.

1. 정적 페이지를 처리한다.
2. `Request`, `Response` 객체를 만들어 Filter객체에 전달한다.
3. `Filter`에서는 인코딩 변환, 보안 처리 등 작업을 한다.
4. WAS에서 `HttpServletRequest`, `HttpServletResponse` 객체로 변환을 하고 `DispatcherServlet`에게 전달한다. (ServletContainer가 servlet의 구현에 대한 부분을 담당하고, 이후 servlet에 요청에 대한 역할을 위임하는 방식 하는 방식)
5. `DispatcherServlet`내의 `Handler Mappling`, `Handler Adapter` 인터페이스 구현체 (spring Container)가 실행되고 controller, service, DB 등의 작업이 처리된다.
6. response에 결과를 넘겨 준다.

모든 접근이 동일한 자원을 공유하면, 하나의 요청이 끝날때 까지 자원을 사용하지 못 하기 때문에, Spring 에서는 ThreadPool을 이용한 Multi Thread를 이용한다.

### Http 요청에 따른 WAS의 Thread 할당 구조

![image](https://user-images.githubusercontent.com/65350890/215467218-b0404a63-ec7a-4178-bd67-34b7fc440c00.png)

요청이 들어올때 `ThreadPool`에서 여유 쓰레드가 있는지 확인하고 요청에 대한 쓰레드를 할당한다. 해당 쓰레드가 servlet을 실행한다. (여유 쓰레드가 없을경우 WAS 정책에 따라 대기하거나 거절할 수 있다.)<br/>

요청이 모두 처리되고 반환되면, 쓰레드는 다시 쓰레디풀에 반환된다. 해당 구조는 쓰레드가 미리 생성되어 쓰레드 생성 및 종료시 발생되는 자원을 절약 할 수 있고, 응답이 빠르다. 또한, 쓰레드에 대한 최대치가 있으므로 처리하기 힘들정도의 요청이 오더라도 기존에 받은 요청에 대해서는 안전하게 처리가 가능하다.
