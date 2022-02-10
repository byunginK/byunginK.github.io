---
layout: post
title: "android basic 3"
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
