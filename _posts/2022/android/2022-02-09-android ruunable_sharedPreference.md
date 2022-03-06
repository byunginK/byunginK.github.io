---
layout: post
title: "android ruunable_sharedPreference"
date: 2022-02-09
categories: [Kotlin]
---

## constraintVertical_bias / constraintHorizontal_bias

보통 `app:layout_constraintEnd_toEndOf="parent"`와`app:layout_constraintStart_toStartOf="parent"`를 사용하여 레이아웃의 중앙에 위치 하도록 한다.

단, bias를 사용하면 수직/수평의 위치에서 한쪽으로 더 비중을 크게 만들 수 있다. 예를 들어 기본 50%대신 30% bias의 값을 설정하게 되면 왼쪽이 더 짧아지게 만들 수 있다. (그림 5 참고)

```xml
<androidx.constraintlayout.widget.ConstraintLayout ...>
    <Button android:id="@+id/button" ...
        app:layout_constraintHorizontal_bias="0.3"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"/>
```

![image](https://user-images.githubusercontent.com/65350890/153204899-ae6e00d6-4011-4d29-b494-bebdf37b65d7.png)

## AppCompatButton

`button`의 하위 클래스로 android 버전에서 앱에 최신 테마를 적용하는데 도움을 준다. 따라서 button의 `background` 및 테마를 적용하게 해준다.

## apply() lazy 선언

지연 초기화를 통해 선언을하고 함수 안에 apply 람다식을 활용하여 numberPicker의 값을 설정하는 방법을 사용

```java
private val numberPicker1: NumberPicker by lazy {
    findViewById<NumberPicker>(R.id.numberPicker1)
        .apply {
            minValue = 0
            maxValue = 9
        }
}
private val numberPicker2: NumberPicker by lazy {
    findViewById<NumberPicker>(R.id.numberPicker2)
        .apply {
            minValue = 0
            maxValue = 9
        }
}
private val numberPicker3: NumberPicker by lazy {
    findViewById<NumberPicker>(R.id.numberPicker3)
        .apply {
            minValue = 0
            maxValue = 9
        }
}
```

## Thema

res파일에 values -> themaes의 xml을 통해 앱의 전체 테마를 스타일 적용 할 수 있다.

```xml
<style name="Theme.Aoppart2chapter03.NoActionBar" parent="Theme.MaterialComponents.DayNight.NoActionBar"/>
```

style을 추가하여 상단에 액션바가 없는 스타일을 상속받아왔고 해당 Thema를 `manifest`의 activity 속성의 thema에 `name`으로 바인딩 해주면 적용된다.

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="fastcampus.aop.part2.chapter03">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.Aoppart2chapter03">
        <activity
            android:name=".MainActivity"
            android:exported="true"
            android:theme="@style/Theme.Aoppart2chapter03.NoActionBar">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        <activity
            android:name=".DiaryActivity"
            android:theme="@style/Theme.Aoppart2chapter03.NoActionBar" />
    </application>

</manifest>
```

## sharedPreference

로컬 DB 저장 또는 파일에 저장하는 방식으로 값을 저장할 수 있으며, `sharedPreference`는 파일에 값을 저장하고 꺼내서 사용할 수 있게 한다.

```java
//파일에 저장을 하여 가져온다. 이름, 모드(다른곳에서 사용 할 수 있는지)
val passwordPreference = getSharedPreferences("password", Context.MODE_PRIVATE)
```

위 코드에서 `"password"`는 name을 의미하며, `Map`형식처럼 이름을 통해 값을 저장하고 가져올 수 있다. 뒤에 Mode는 해당 `sharedPreference`의 접근에대한 권한을 설정한다.
(private)는 해당 패키지에서만 접근 가능

### 저장

`sharedPreference`를 저장하는 방식은 아래와 같다.

```java
passwordPreference.edit {
    putString("password",passwordFromUser)
    commit()
}
```

`putString()`에서 첫번째 인자는 `name`을 넣고 두번째 인자는 값을 넣는다. 그리고 `commit()`함수를 무조건 실행해줘야 저장이 된다.

```java
passwordPreference.edit(commit = true) {
    putString("password",passwordFromUser)
}
```

위의 방식처럼 `commit=ture`를 통해 람다식이 종료되면 무조건 commit을 실행하는 방식을 통해서도 구현이 가능하다.

### 값 가져오기

값을 가져올때는 `name`으로 가져오고 default값을 설정해주어야 한다.

```java
passwordPreference.getString("password","000")
```

## Alert

알림 팝업창을 띄우며 다양한 확장함수를 통해 값을 설정할 수 있다. 특히 `positiveButton`은 두개의 인자를 받는 람다식으로 클릭시 실행할 행동을 추가할 수 있다.

```java
private fun showErrorAlertDialog(){
    AlertDialog.Builder(this)
        .setTitle("실패")
        .setMessage("비밀번호가 잘못되었습니다.")
        .setPositiveButton("확인"){ _, _ -> } //두개의 인자를 받음
        .create()
        .show()
}
```

## runnable

Main UI를 작동하는 쓰레드 이외에 별도의 쓰레드를 생성
`Runnable{}`인터페이스 람다식을 이용하여 입력한 텍스트를 `sharedPreferences`에 저장하도록 구현

```java
//별도 쓰레드로 잠깐 멈췄을때 저장
val runnable = Runnable {
    getSharedPreferences("diary",Context.MODE_PRIVATE).edit {
        putString("detail", diaryEditText.text.toString())
    }
    Log.d("DiaryActivity", "SAVE ${diaryEditText.text.toString()}")
}
```

## addTextChangedListener

텍스트가 변경될 때마다 이벤트가 작동하도록하는 리스너이다.
`diaryEditText`라는 `EditText`에서 텍스트가 변경될때마다 저장되도록 기능 구현.

**`Handler`를 통해 main쓰레드와 runnable쓰레드를 연결하도록한다.**'
Handler를 사용하는 이유는 비동기적(Asynchronous)으로 어떤 Job을 처리하기 위해.
※ Activity의 Main thread는 UI thread라고 불리며, UI를 처리할 때 시간이 오래 걸리는 작업을 수행하면 화면이 버벅이거나 ANR(Android Not Responding)과 같은 문제가 발생.
Handler는 Looper라는 객체를 통해 동작한다. 1개의 Thread에 1개의 Looper가 있다.

```java
//main UI쓰레드와 별도 쓰레드를 핸들러를 통해서 연결
private val handler = Handler(Looper.getMainLooper())

//텍스트가 바뀔때마다 이벤트 작동
diaryEditText.addTextChangedListener {
    Log.d("DiaryActivity", "TextCahnged::$it")
    handler.removeCallbacks(runnable) //이전에 있는 변화를 우선 지운다
    handler.postDelayed(runnable, 500)
}
```

```
diaryEditText 텍스트 변경 -> addTextChangedListener 이벤트 작동 -> main 쓰레드에서 removeCallbacks을 통해 0.5초 이전에 작성된 텍스트는 저장하지 않도록 제거 -> 만약 0.5초가 지나면 저장 (0.5초 지연은 postDelayed()를 통해 진행)
```
