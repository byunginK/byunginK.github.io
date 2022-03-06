---
layout: post
title: "android constraintlayout"
date: 2022-02-07
categories: [Kotlin]
---

## constraintlayout

플랫 뷰 계층 구조(중첩 뷰 그룹이 없음)로 크고 복잡한 레이아웃을 만들 수 있다.

Default 레이아웃으로 설정되어 있으며, `androidx.constraintlayout.widget.ConstraintLayout`으로 설정 가능하다.

```xml
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">
</androidx.constraintlayout.widget.ConstraintLayout>
```

`app:layout_constraintStart_toStartOf`같은 형식으로 사용되며 첫번째 지시자는 시작 요소의 포지션, 두번째 지시는 다음 요소의 포지션을 의미한다.

1. top = 위
2. bottom = 아래
3. start = 왼쪽
4. end = 오른쪽

`app:layout_constraintTop_toTopOf`를 예시로 살펴보면 `constraintTop`은 시작 요소의 위쪽과 `toTopof`는 다음요소의 포지션(위쪽)을 연결한다는 의미이다.

### 추가 속성

내부 요소의 정렬을 위치한다.

```xml
<TextView
    android:id="@+id/textView1"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_margin="5dp"
    android:text="1"
    android:gravity="center"
    android:textColor="@color/white"
    android:background="@drawable/circle_blue"
    android:visibility="gone"
    tools:visibility="visible"
    android:textSize="18sp"
    android:textStyle="bold" />
```

- `android:gravity="center"`은 가운데 정렬을 해준다.
- `android:background="@drawable/circle_blue"`은 drawable소스를 가져다 배경으로 설정하며, 해당 레이아웃에서만 미리 보고 추후 kotiln에서 값을 재설정 할 수 있다.
- `android:visibility="gone"`은 내용을 숨긴다.

```xml
<Button
    android:id="@+id/addButton"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_marginTop="10dp"
    android:layout_marginEnd="16dp"
    android:text="@string/addNumber"
    app:layout_constraintEnd_toStartOf="@id/clearButton"
    app:layout_constraintHorizontal_chainStyle="packed"
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintTop_toBottomOf="@id/numberPicker" />
<!--constraintHorizontal_chainStyle은 사이 공간의 비율을 설정-->
```

`app:layout_constraintHorizontal_chainStyle="packed"` 은 수평의 연결된 스타일을 어떻게 할지 설정한다.

## MainActivity

### by lazy

지연 초기화로 선언만 해놓고 실제 사용시 초기화되어 사용된다.

```java
private val clearButton: Button by lazy {
    findViewById<Button>(R.id.clearButton)
}

private val addButton: Button by lazy {
    findViewById<Button>(R.id.addButton)
}

private val runButton: Button by lazy {
    findViewById<Button>(R.id.runButton)
}

private val numberPicker: NumberPicker by lazy {
    findViewById<NumberPicker>(R.id.numberPicker)
}

private val numberTextViewList: List<TextView> by lazy {
    listOf<TextView>(
        findViewById<TextView>(R.id.textView1),
        findViewById<TextView>(R.id.textView2),
        findViewById<TextView>(R.id.textView3),
        findViewById<TextView>(R.id.textView4),
        findViewById<TextView>(R.id.textView5),
        findViewById<TextView>(R.id.textView6)
    )
}
```

### numberPicker

kotlin에서 numberPicker의 최소값과 최대값을 설정할 수 있다.

```java
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)

    numberPicker.minValue = 1
    numberPicker.maxValue = 45

    initRunButton()
    initAddButton()
    initClearButton()
}
```

### initRunButton()

6개의 랜덤 숫자를 생성하여 list를 가지고 `textView`에 맵핑하여 표시한다.

```java
private fun initRunButton() {
    runButton.setOnClickListener {
        val list = getRandomNumber()

        didRun = true

        list.forEachIndexed { index, number ->
            val textView = numberTextViewList[index]
            textView.text = number.toString()
            textView.isVisible = true

            setNumberBackground(number,textView)
        }
        Log.d("MainActivity", list.toString())
    }
}
```

### getRandomNumber

6개의 랜덤 숫자를 list에 담아서 반환한다.

#### apply()

반환값이 없으며, `this`로 앞 객체를 받아서 적용 및 설정할 수 있다. 따라서 보통 객체의 초기화 값을 할때 이용된다. 아래 코드에서는 `mutableListOf<Int>`를 `this`로 받아서 값을 `add()`하는것을 볼 수 있다.

```java
private fun getRandomNumber(): List<Int> {
    val numberList = mutableListOf<Int>()
        .apply {
            for (i in 1..45) {
                if(pickNumberSet.contains(i)){
                    continue
                }
                this.add(i)
            }
        }
    numberList.shuffle()

    return pickNumberSet.toList() + numberList.subList(0, 6 - pickNumberSet.size).sorted()
}
```

### setNumberBackground

숫자 크기에 맞게 drawable의 소스를 맵핑하여 textView에 설정해준다.

```java
private fun setNumberBackground(number: Int, textView: TextView){
    when(number){
        //안드로이드상 소스를 가져오는 것으로 contextCompat 메소드를 통해 소스를 가져오고 첫번째
        //인자로는 현재 context이므로 this를 넣어주고 두번째 인자는 적용시킬 drawable소스를 넣어준다.
        in 1..10 -> textView.background = ContextCompat.getDrawable(this,R.drawable.circle_yellow)
        in 11..20 -> textView.background = ContextCompat.getDrawable(this,R.drawable.circle_blue)
        in 21..30 -> textView.background = ContextCompat.getDrawable(this,R.drawable.circle_red)
        in 31..40 -> textView.background = ContextCompat.getDrawable(this,R.drawable.circle_gray)
        else -> textView.background = ContextCompat.getDrawable(this,R.drawable.circle_green)


    }
}
```

### initAddButton

번호 추가 버튼에 대한 기능이며, numberPicker에서 번호를 set에 담아준다.

```java
private fun initAddButton() {
    addButton.setOnClickListener {
        if (didRun) {
            Toast.makeText(this, "초기화 후에 시도해주세요.", Toast.LENGTH_SHORT).show()
            return@setOnClickListener
        }
        if (pickNumberSet.size >= 5) {
            Toast.makeText(this, "번호는 5개까지만 선택할 수 있습니다.", Toast.LENGTH_SHORT).show()
            return@setOnClickListener
        }
        if (pickNumberSet.contains(numberPicker.value)) {
            Toast.makeText(this, "이미 선택한 번호 입니다.", Toast.LENGTH_SHORT).show()
            return@setOnClickListener
        }
        val textView = numberTextViewList[pickNumberSet.size]
        textView.isVisible = true
        textView.text = numberPicker.value.toString()

        setNumberBackground(numberPicker.value, textView)

        pickNumberSet.add(numberPicker.value)
    }
}
```

### initClearButton

set을 클리어하여 초기화한다.

```java
private fun initClearButton() {
    clearButton.setOnClickListener {
        pickNumberSet.clear()
        numberTextViewList.forEach {
            it.isVisible =false
        }
        didRun = false
    }
}
```

## 결과

![lotto](https://user-images.githubusercontent.com/65350890/153207641-beb5bf04-bc7b-4dca-b38f-6e4f28c1d00d.gif)
