---
layout: post
title: "android basic 9"
date: 2022-02-22
categories: [Kotlin]
---

# Firebase messaging

### 설정

프로젝트 수준에서 아래와 같이 추가해준다.

```gradle
plugins {
    id 'com.google.gms.google-services' version '4.3.10'
}
```

앱 수준에서 코드를 추가해준다.

```gradle
plugins {
    id 'com.google.gms.google-services'
}

dependencies {

    implementation platform('com.google.firebase:firebase-bom:29.1.0')
    implementation 'com.google.firebase:firebase-messaging-ktx'
}
```

또한 가이드에 따라 google.service.json파일을 프로젝트수준에서 추가를 해준다.

그리고 아래와 같이 클래스를 만들어 준다.

```java
class MyFirebaseMessagingService: FirebaseMessagingService() {

    //토큰이 갱신될때마다 서버에 해당 토큰을 업데이트 해주어야 한다.
    //(현재 프로젝트는 사용 안함)
    override fun onNewToken(p0: String) {
        super.onNewToken(p0)
    }

    //메세지를 받는 부분
    override fun onMessageReceived(message: RemoteMessage) {
        super.onMessageReceived(message)
    }
}
```

그리고 메세지 이벤트가 발생했을때 서비스를 이용한다는 설정을 manifest에도 해주어야 한다.

```xml
<application
    android:allowBackup="true"
    android:icon="@mipmap/ic_launcher"
    android:label="@string/app_name"
    android:roundIcon="@mipmap/ic_launcher_round"
    android:supportsRtl="true"
    android:theme="@style/Theme.Chapter09">

    ...


    <!--MESSAGING_EVENT이벤트 발생시 MyFirebaseMessagingService를 이용하겠다는 의미 -->
    <service android:name=".MyFirebaseMessagingService"
        android:exported="false"><!--해당 설정은 서비스를 공유와 같은것에 사용할지 결정-->
        <intent-filter>
            <action android:name="com.google.firebase.MESSAGING_EVENT"/>
        </intent-filter>
    </service>
</application>
```

그리고 Firebase가 처음 실행이되면 고유 토큰값을 제공하는데 이는 메세지를 알림이나 메세지를 받을때 필요하다. 토큰을 받는 방법은 아래와 같이 앱 실행시 받도록 하였다.

```java
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)
    initFirebase()

}
private fun initFirebase() {
    FirebaseMessaging.getInstance().token
        .addOnCompleteListener {
            if(it.isSuccessful){

                firebaseTokenTextView.text = it.result
            }
    }
}
```

해동 토큰은 위에 `MyFirebaseMessagingService`에서 `onNewToken`함수에서 새로 갱신될때 서버에서 업데이트하는 로직을 구성할 수도 있다.

### 알림

안드로이드 8.0부터는 알림채널을 설정해야한다.

Notification으로 메세지를 받았을때와 채널 등 설정을 생성해준다.
아래 코드는 원래 토큰이 새로 발급 될때마다 처리해주는(디비저장등)로직을 구현해야하나 이번에는 구현하지 않았다.

```java
//토큰이 갱신될때마다 서버에 해당 토큰을 업데이트 해주어야 한다.
//(현재 프로젝트는 사용 안함)
override fun onNewToken(p0: String) {
    super.onNewToken(p0)
}
```

**notification channel 생성**

```java
private fun createNotificationChannel() {
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
        val channel = NotificationChannel(
            CHANNEL_ID,
            CHANNEL_NAME,
            NotificationManager.IMPORTANCE_DEFAULT
        )
        channel.description = CHANNEL_DESCRIPTION

        (getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager)
            .createNotificationChannel(channel)
    }
}
```

**타입별 알림 제목, 내용, 중요도를 설정**

```java
private fun createNotification(
    type: NotificationType,
    title: String?,
    message: String?
): Notification {
    //인텐드로 메인 화면에 넘기기 위해 생성
    val intent = Intent(this, MainActivity::class.java).apply {
        putExtra("notificationType", "${type.title} 타입")
        //기존화면을 갱신하는 flag (작업 및 스택)
        addFlags(Intent.FLAG_ACTIVITY_SINGLE_TOP)
    }

    //PendingIntent 인텐트를 다룰수 있는 권한을 준다.
    val pendingIntent = PendingIntent.getActivity(this, type.id, intent, PendingIntent.FLAG_UPDATE_CURRENT)

    val notifiactionBuilder = NotificationCompat.Builder(this, CHANNEL_ID) // context, id
        .setSmallIcon(R.drawable.ic_notification) // 알림 아이콘
        .setContentTitle(title) // 알림 제목
        .setContentText(message) // 알림 내용
        .setPriority(NotificationCompat.PRIORITY_DEFAULT) // 알림 중요도
        .setContentIntent(pendingIntent) //인텐드를 담아 알림에 적용 (알림 클릭시 인텐드의 내용도 같이 넘어감)
        .setAutoCancel(true) //클릭시 자동으로 닫아줌


    when (type) {
        NotificationType.NORMAL -> Unit
        NotificationType.EXPANDABLE -> {
            notifiactionBuilder.setStyle(
                NotificationCompat.BigTextStyle()
                    .bigText(
                        "..."
                    )
            )
        }
        NotificationType.CUSTOM -> {
            notifiactionBuilder
                .setStyle(NotificationCompat.DecoratedCustomViewStyle())
                .setCustomContentView(
                    RemoteViews(
                        packageName,
                        R.layout.view_custom_notification //layout을 별도로 만들어 사용할 수 있다.
                    ).apply {
                        setTextViewText(R.id.title, title)
                        setTextViewText(R.id.message, message)
                    }
                )
        }
    }

    return notifiactionBuilder.build()
}
```

intent를 생성 및 알림에 담아 넘겨주면 알림을 클릭했을때 intnet의 값들을 알림에서 클릭했을때 사용할 수 있다.
intent의 `addFlags`프로퍼티는 화면의 스택을 설정할 수 있다.
[자세히 살펴보기](https://developer.android.com/guide/components/activities/tasks-and-back-stack?hl=ko)

**ovveride를 통해 메세지를 받는 부분을 구현 할 수 있다.**

```java
//메세지를 받는 부분
override fun onMessageReceived(remoteMessage: RemoteMessage) {
    super.onMessageReceived(remoteMessage)

    createNotificationChannel()

    val type = remoteMessage.data["type"]?.let {
        NotificationType.valueOf(it)
    }
    val title = remoteMessage.data["title"] //fireBase에서 제목으로 넘긴걸 받음
    val message = remoteMessage.data["message"] // fireBase에서 메시지

    type ?: return



    NotificationManagerCompat.from(this)
        .notify(type.id, createNotification(type, title, message))
}
```

위의 `NotificationType`은 Enum으로 별도 생성한 클래스

`MainActivity`에서 `onNewIntent`메소드를 통해 intent가 새로 생성되면 구현되느 로직을 작성한다.

이번 프로젝트에서는 새로intent가 생성이되면 화면에 알림의 내용을 text로 업데이트하도록 하였다.

```java
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)
    initFirebase()
    updateResult()
}

override fun onNewIntent(intent: Intent?) {
    super.onNewIntent(intent)

    setIntent(intent)
    updateResult(true)
}

@SuppressLint("SetTextI18n")
private fun updateResult(isNewIntent: Boolean = false){
    resultTextView.text = (intent.getStringExtra("notificationType")?:"앱 런처") +
    if(isNewIntent){
        "(으)로 갱신했습니다."
    }else{
        "(으)로 실행 했습니다."
    }
}
```

[전체 코드 보러 가기](https://github.com/byunginK/Andriod_Project/tree/main/chapter09)
