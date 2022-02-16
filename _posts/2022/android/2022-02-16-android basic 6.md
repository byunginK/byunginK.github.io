---
layout: post
title: "android basic 6"
date: 2022-02-16
categories: [Kotlin]
---

# UI

### ChainStyle

한 라인에서 요소의 가장 첫번째 속성에 설정 할 수 있다. 요소들의 간격을 조절할 수 있다.
아래 이미지 참고

![image](https://user-images.githubusercontent.com/65350890/154275590-4206d0ec-9708-4661-8e73-62d820e3a418.png)

### seekBar

손가락으로 눌러서 조절 할 수 있는 progress bar를 표현 할 수 있다.

```xml
<SeekBar
    android:id="@+id/seekBar"
    android:layout_width="0dp"
    android:layout_height="wrap_content"
    android:layout_marginHorizontal="20dp"
    android:max="60"
    android:progressDrawable="@color/transparent"
    android:thumb="@drawable/ic_thumb"
    android:tickMark="@drawable/drawable_tick_mark"
    app:layout_constraintBottom_toBottomOf="parent"
    app:layout_constraintLeft_toLeftOf="parent"
    app:layout_constraintRight_toRightOf="parent"
    app:layout_constraintTop_toBottomOf="@id/remainMinutesTextView" />
```

위의 소스코드에서 thumb은 현재 값을 표시, tickmark는 값의 하나하나 위치를 표시해준다.

### themes

안드로이드는 처음 OS에서 테마를 뿌려주고 그위에 view 스타일을 바인딩한다. 따라서 처음에 테마의 background를 지정하면 첫 앱 스타트 시 배경색이 하얗게 되었다가 변경되는것을 처음부터 원하는 색으로 설정할 수 있다.

```xml
<!-- themes.xml-->
<resources xmlns:tools="http://schemas.android.com/tools">
    <!-- Base application theme. -->
    <style name="Theme.Chapter06" parent="Theme.MaterialComponents.DayNight.NoActionBar">
        <!-- Status bar color. -->
        <item name="android:statusBarColor">@color/pomodoro_red</item>
        <!-- Customize your theme here. -->
        <item name="android:windowBackground">@color/pomodoro_red</item>
    </style>
</resources>
```

---

# Back

### CountDownTimer

`CountDownTimer`를 상속받아 override하여 사용 한다.
인자로는 long타입의 시작 숫자와 카운트다운할 인터벌 설정 값을 준다.

```java
private fun createCountDownTimer(initialMillis: Long) =
    object : CountDownTimer(initialMillis, 1000L) {
        override fun onTick(p0: Long) {
            updateRemainTime(p0) //유닛 하나가 실행될때 인자로 알려줌
            updateSeekBar(p0)
        }

        override fun onFinish() {
            completeCountDown()

        }

    }

//남은 시간을 받아서 textview에 표시
private fun updateRemainTime(remainMills: Long) {

    val remainSeconds = remainMills / 1000

    remainMinutesTextView.text = "%02d'".format(remainSeconds / 60)
    remainSecondsTextView.text = "%02d".format(remainSeconds % 60)
}

//seekBar에 남은 시간을 표시
private fun updateSeekBar(remainMills: Long) {
    seekBar.progress = (remainMills / 1000 / 60).toInt()
}

//카운트 다운 완료
private fun completeCountDown() {
    updateRemainTime(0)
    updateSeekBar(0)

    soundPool.autoPause()
    //let 람다식을 활용하여 null safeCall
    bellSoundId?.let {
        soundPool.play(it, 1F, 1F, 0, 0, 1F)
    }
}
```

### SeekBar

progress Bar에서 사용자가 설정한 값이 변경 될때마다 Listener로 전달 받는다.

```java
private fun bindViews() {
    seekBar.setOnSeekBarChangeListener(
        object : SeekBar.OnSeekBarChangeListener {
            //변경 될때 마다 값을 textView에 표시
            override fun onProgressChanged(
                seekBar: SeekBar?,
                progress: Int,
                fromUser: Boolean
            ) {
                if (fromUser) {
                    updateRemainTime(progress * 60 * 1000L)
                }
            }
            //bar를 조절할때 countDown하지 않도록 함
            override fun onStartTrackingTouch(seekBar: SeekBar?) {
                stopCountDown()
            }
            //bar 조절을 멈췄을때 카운트 다운 시작 및 0일때 체크
            override fun onStopTrackingTouch(seekBar: SeekBar?) {
                seekBar ?: return
                if(seekBar.progress == 0){
                    stopCountDown()
                }else {
                    startCountDown()
                }
            }
        }
    )
}

private fun stopCountDown(){
    currentCountDownTimer?.cancel()
    currentCountDownTimer = null
    soundPool.autoPause()
}

private fun startCountDown(){
    currentCountDownTimer = createCountDownTimer(seekBar.progress * 60 * 1000L)
    currentCountDownTimer?.start()

    tickingSoundId?.let {
        soundPool.play(it, 1F, 1F, 0, -1, 1F)
    }
}
```

### soundPool

메모리에 작은 음악 리소스를 사용하여 소리를 내게 해준다.

```java
private fun initSounds() {
    tickingSoundId = soundPool.load(this, R.raw.timer_ticking, 1)
    bellSoundId = soundPool.load(this, R.raw.timer_bell, 1)
}
```

id를 부여하기 위해 파일을 load하여 선언 한다.

소리를 play 하기 위해서는 아래와같이 설정하며 인자는 soundId, 왼쪽, 오른쪽 소리 크기, 스트림 우선순위, 반복, 재생속도를 의미한다.

```java
tickingSoundId?.let {
    soundPool.play(it, 1F, 1F, 0, -1, 1F)
}
```

또한 Activity LifeCycle에 맞추어 pause와 release해준다.

```java
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)

    bindViews()
    initSounds()
}
//화면 다시 켰을때
override fun onResume() {
    super.onResume()
    soundPool.autoResume()
}

//앱이 화면에 안보일때
override fun onPause() {
    super.onPause()
    soundPool.autoPause()
}

//메모리에서 해제
override fun onDestroy() {
    super.onDestroy()
    soundPool.release()
}
```

현재 프로젝트에서는 `autoPause()`, `autoResume()`을 하였지만, 각각 ID를 통해 컨트롤 할 수 있다.
