---
layout: post
title: "JVM 메모리 leak 분석"
date: 2023-06-02
categories: [JAVA]
---
___

자바의 경우 JVM 위에서 돌아가기에 메로리 누수의 추적이 다른 언어에 비해 쉬운 편이다

우선 해당 문제의 PC에서 heap memory dump를 떠서 살펴보면 어떤 오브젝트에서 많은 메모리가 사용되었는지 확인이 가능하다

### 1\. memery leak이 발생 중인 프로세스 아이디 획득

jps 명령어를 이용할 수 있고 리눅스라면 ps 명령어로도 가능하다

![](https://blog.kakaocdn.net/dn/liIfT/btrxoGIROnn/G1KJx15q9h4gwmJjXWoOZ1/img.png)

### 2\. jmap을 이용한 메모리덤프 획득

기본 java 설치시 jmap도 설치되어있는데 이를 이용하여 덤프를 뜬다

```
jmap -dump:format=b,file=heap.hprof [process ID]
```

커맨드에서 위 명령어 입력 시 아래와 같이 진행된다 

![](https://blog.kakaocdn.net/dn/bkBBUX/btrxwZnJyf0/Or71Mq2qCPh3TGLObkJviK/img.png)

프로세스아이디 17152의 힙 덤프 생성

그리고 커맨드를 입력한 경로에 보면 heap.hprof 파일이 생성되어 있을 것이다

### 3\. Eclipse MAT을 이용한 분석 

[https://www.eclipse.org/mat/](https://www.eclipse.org/mat/)

위 링크에서 무료 툴인 MAT을 다운로드하여 실행한다

![](https://blog.kakaocdn.net/dn/dbRQhT/btrxmzEw6hD/tCsI1vwW9tQmRarBGsWRz0/img.png)

File -> Open Heap Dump.. 를 눌러 추출했던 heap.hprof를 선택해 주면 된다

이때 추출할 PC의 메모리가 상당히 많이 사용된다 문제의 프로세스에서 사용한 만큼의 램이 필요하며 부족할 경우 분석 중 에러가 뜨게 된다  
이 메모리 사용량을 제한하기 위해 MAT에서 기본 설정이 1024MB로 설정되어 있는데 자신의 PC에서 충분한 램을 가지고 있다면 MAT설치 경로에 MemoryAnalyer.ini를 열어서 가장 아랫부분의 제한용량을 수정해 준다

![](https://blog.kakaocdn.net/dn/cCm8Hw/btrxwAhpn2L/w9PobtRKW9IH225ldzKGY0/img.png)

원래는 해당 블럭이 -Xmx1024m 로 되어있었음

### 4\. 결과 확인 & 원인 탐색

잠시 기다리면 분석이 완료되고 

![](https://blog.kakaocdn.net/dn/bN75mD/btrxrcPnOX9/PYhFTIhn90LkkuESlBMKuK/img.png)

이와 같이 결과가 발생한다 나의 경우엔 테스트 프로그램으로 ArrayList를 무한히 증가시키는 프로그램을 만들어 돌렸었는데 위와 같이 arraylist에서 메모리의 사용량이 99% 사용량도 3기가를 넘겨버렸다 

이제 이로 추측하여 해당 리스트를 증가시키는 부분을 추적하고 관련 코드를 수정해볼 수 있다

___

※추가로 제대로 분석해보진 않았지만 java실행시 명령어를 통해서 GC로그를 남길 수 있는방법도 있었다

```
java Xms2g -Xmx4g -XX:+UseG1GC -XX:NewRatio=7 -XX:+PrintGCDetails -verbose:gc -Xloggc:./log/gc.log -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=./log -jar [jar파일명]
```

위와같이 java와 설정들을 넣어주고 맨끝에 -jar \[실행파일명\]을 넣어주면된다 

참조 : [https://techblog.woowahan.com/2628/](https://techblog.woowahan.com/2628/)

___