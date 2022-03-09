---
layout: post
title: "android pedingIntent"
date: 2022-02-27
categories: [Kotlin]
---

# Back

### PendingIntent

Pending 은 '보류', '임박' 이런 뉘앙스를 갖고 있다. PendingIntent 는, 가지고 있는 Intent 를 당장 수행하진 않고 특정 시점에 수행하도록 하는 특징을 갖고 있다. 이 '특정 시점'이라 함은, 보통 해당 앱이 구동되고 있지 않을 때이다.

PendingIntent는 생성자가 없고 다음 세 개의 메소드들에 의해서 객체가 생성

자세한 내용은 해당 링크 참고 : (https://velog.io/@haero_kim/Android-PendingIntent-%EA%B0%9C%EB%85%90-%EC%9D%B5%ED%9E%88%EA%B8%B0)

추가 참고 링크 : (https://developer-joe.tistory.com/22)

### BroadcastReceiver

Android 앱은 Android 시스템 및 기타 Android 앱에서 게시-구독 디자인 패턴과 유사한 브로드캐스트 메시지를 받거나 보낼 수 있습니다.

예를 들어 Android 시스템은 시스템 부팅 또는 기기 충전 시작과 같은 다양한 시스템 이벤트가 발생할 때 브로드캐스트를 전송
[안드로이드 개발자 문서 보기](https://developer.android.com/guide/components/broadcasts?hl=ko)

[전체 코드 보러 가기](https://github.com/byunginK/Andriod_Project/tree/main/chapter11)
