---
layout: post
title: "android basic 1"
date: 2022-02-06
categories: [Kotlin]
---

## LinearLayout

수직, 수평으로 층으로 레이아웃을 하는 방식

`android:orientation`으로 수직으로 정렬할지 수평으로 정렬할지 결정 추가록 padding에 들어간 dp는 안드로이드 디바이스의 고정값으로 쓰여진다.

- 예제

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp"
    tools:context=".MainActivity">
    <!--안드로이드는 dp를 사용 (디바이스픽셀)-->
</LinearLayout>
```

## TextView

텍스트를 표시할 수 있게 해준다. 폰트 사이즈, 굵기 등을 설정할 수 있다.

_layout_width와 layout_height는 아래 주석과 같이 설정 할 수 있다_

_글자 크기의 경우 개인마다 시스템에서 폰트의 크기를 조정할 수 있으므로 `sp`를 사용하여 비율로 조정될 수 있게 설정_

```xml
    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="@string/height"
        android:textColor="@color/custom_black"
        android:textSize="20sp"
        android:textStyle="bold" />
    <!--wrap_content:  컨텐츠의 높이만큼
        match_parent: 부모의 넓이 만큼 (여기서는 기기 꽉 채운다는 의미)
        폰트 sp는 각 폰의 환경설정에 크기에 적용된 비율에 적용된다 (dp는 고정값).
    -->
```

## EditText

텍스트를 입력 할 수 있는 공간을 배치한다. `inputType`으로 텍스트의 종류를 선택 할 수 있다.

```xml
    <EditText
        android:id="@+id/heightEditText"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="10dp"
        android:inputType="number" />
    <!--텍스트를 추가할 수 있는 영역
        inputType은 클릭시 입력패드의 종류를 선택할 수있게함
    -->
```

### values

레이아웃 xml에 String, color등의 값을 직접 넣어 줄 수 있지만 values에서 설정 후 끌어 쓸 수 있다. (ex : `android:text"@string/height"`)

1. strings.xml

```xml
<resources>
    <string name="app_name">aop-part2-chapter01</string>
    <string name="height">신장</string>
    <string name="weight">체중</string>
    <string name="confirm">확인하기</string>
</resources>
```

### tools

TextView의 값에 kotlin에서 값을 받아 넣어줘야할 경우 초기값을 빈값으로 했을때 레이아웃 잡기 어렵다. 그때 tools 속성을 import하여 사용하면 레이아웃에서는 보이지만, 실제 앱에서는 빈 값으로 나온다.

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    xmlns:tools="http://schemas.android.com/tools">

    <TextView
        android:id="@+id/resultTextView"
        tools:text="과체중입니다"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />
</LinearLayout>
```

## MainActivity

안드로이드에서 Activity는 한 화면을 의미하며, Main은 첫 화면을 뜻한다. `AppCompatActivity()`를 상속받아 `onCreat()`를 통해 화면을 생성한다. `setContentView(R.layout.activity_main)`은 어떤 레이아웃을 사용할지 설정 해준다.

```java
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
    }
}
```

### findViewById()

서로 맵핑을 id를 통해 하며 layout xml에서도 각 태그에게 `android:id`를 부여하고 `findViewById()`에서도 인자로 `android:id`를 넣어주면 값을 가져온다.

**activity_main.xml**

```xml
...
<EditText
    android:id="@+id/heightEditText"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_marginTop="10dp"
    android:inputType="number" />

<EditText
    android:id="@+id/weightEditText"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_marginTop="10dp"
    android:ems="10"
    android:inputType="number" />

<Button
    android:id="@+id/resultButton"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_marginTop="30dp"
    android:text="@string/confirm" />
...
```

**MainActivity.kt**

```java
val heightEditText: EditText = findViewById(R.id.heightEditText)
val weightEditText = findViewById<EditText>(R.id.weightEditText)
//버튼 매핑
val resultButton = findViewById<Button>(R.id.resultButton)
```

### setOnClickListener()

버튼 클릭시 실행되는 이벤트 리스너로 람다식으로 로직을 추가하여 버튼 클릭시 작동 되게 한다.

```java
resultButton.setOnClickListener {
    Log.d("MainActivity","ResultButton click")

    //만약 빈값을 그대로 받게되면 아래 toInt() 부분에서 값이 없어서 에러 발생
    if(heightEditText.text.isEmpty() || weightEditText.text.isEmpty()){
        Toast.makeText(this, "빈 값이 있습니다.",Toast.LENGTH_SHORT).show()
        //@setOnClickListener 라벨을 통해 return할 부분 지정
        return@setOnClickListener
    }

    val height: Int = heightEditText.text.toString().toInt()
    val weight: Int = weightEditText.text.toString().toInt()

    Log.d("MainActivity","height : $height , weight : $weight")
}
```

## AndroidManifest.xml

안드로이드에서 activity를 추가하게되면 `Manifest.xml`에 activity태그를 추가하여 맵핑을 해주어야 한다.

예제로 `ResultActivity.kt`를 추가하였고 `Manifest.xml`에도 적용 시켜주었다.

```xml
<activity
    android:name=".MainActivity"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />

        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
<activity android:name=".ResultActivity"/>
```

## Intent

메시징 객체로 화면간 또는 통신시 값을 전달하는 용도로 사용된다.

**인텐트 전송 흐름**
![image](https://user-images.githubusercontent.com/65350890/152677124-1d9a3d77-3170-47b7-ba33-4ef3ff51d60c.png)

**예제코드**

```java
//Intent에 다음 열릴 엑티비티 자바 파일을 담고 startActivity()로 시작을 해준다.
//안드로이드 시스템으로 intent를 넘길때 값을 넘기고
//받는쪽 activity에서 onCreate()할 때 intent에 포함된 값을 받을 수 있다.
val intent = Intent(this, ResultActivity::class.java)
intent.putExtra("height",height)
intent.putExtra("weight",weight)

startActivity(intent)
```

`ResultActivity.kt`에서는 아래와 같이 값을 받을 수 있다.

```java
class ResultActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_result)
        val height = intent.getIntExtra("height", 0)
        val weight = intent.getIntExtra("weight", 0)
    }
}
```

## 결과

![ezgif com-gif-maker](https://user-images.githubusercontent.com/65350890/153207050-8b453698-c88d-40c2-8bc5-275195a4ef5d.gif)
