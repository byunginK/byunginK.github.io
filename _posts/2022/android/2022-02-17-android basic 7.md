---
layout: post
title: "android basic 7"
date: 2022-02-17
categories: [Kotlin]
---

# UI

## 상태를 통한 아이콘 변경

재생, 녹음, 중지등의 상태에 따라 화면 아이콘이 변경되는데 이것을
kotiln으로 값에 따라 변경되도록 구성

● 상태들을 Enum으로 구성

```java
enum class State {
    BEFORE_RECORDING,
    ON_RECORDING,
    AFTER_RECORDING,
    ON_PLAYING
}
```

● 이미지를 상태에 따라 set하는 로직 구성

```java
//AppCompatImageButton 안드로이드 호환성 문제로 다른 버전에도 적용하기위한 클래스 상속
class RecordButton(context: Context, attrs: AttributeSet): AppCompatImageButton(context, attrs) {
    fun updateIconWithState(state: State){
        when(state){
            State.BEFORE_RECORDING -> {
                setImageResource(R.drawable.ic_record)
            }
            State.ON_RECORDING -> {
                setImageResource(R.drawable.ic_stop)
            }
            State.AFTER_RECORDING -> {
                setImageResource(R.drawable.ic_play)
            }
            State.ON_PLAYING -> {
                setImageResource(R.drawable.ic_stop)
            }
        }
    }
}
```

● MainActivity에서 실행되는 메소드에 따라 상태 변경

```java
//위에서 설정한 버튼 클래스를 상속
private val recordButton: RecordButton by lazy {
    findViewById(R.id.recordButton)
}

private var state = State.BEFORE_RECORDING
    set(value) {
        field = value
        resetButton.isEnabled = (value == State.AFTER_RECORDING) || (value == State.ON_PLAYING)
        recordButton.updateIconWithState(value)
    }

private fun bindViews() {
    recordButton.setOnClickListener {
        when (state) {
            State.BEFORE_RECORDING -> {
                startRecording()
            }
            State.ON_RECORDING -> {
                stopRecording()
            }
            State.AFTER_RECORDING -> {
                startPlaying()
            }
            State.ON_PLAYING -> {
                stopPlaying()
            }
        }
    }
}
```

각 메서드가 실행되면서 `stat` 변수 값이 변경되고 `set`을 통해
`recordButton.updateIconWithState(value)`으로 이미지 변경

### Custom Drawing

● Paint 클래스: Paint 클래스는 기하 도형, 텍스트 및 비트맵을 그리는 방법에 대한 스타일 및 색상 정보를 보유
● onDraw메소드 : onDraw()의 매개변수는 뷰에서 자기 자신을 그릴 때 사용할 수 있는 Canvas 객체입니다. Canvas 클래스는 텍스트, 선, 비트맵 및 기타 여러 그래픽 프리미티브를 그리기 위한 메서드를 정의

예를 들어, Canvas는 선을 그릴 수 있는 메서드를 제공하고 Paint는 선의 색상을 정의하는 메서드를 제공합니다. Canvas에는 직사각형을 그리는 메서드가 있는 반면 Paint는 직사각형을 색으로 채울지 또는 비워 둘지 정의합니다. 간단히 말해, Canvas는 화면에 그릴 수 있는 도형을 정의하고, Paint는 그리는 각 도형의 색상, 스타일, 글꼴 등을 정의

```java
override fun onSizeChanged(w: Int, h: Int, oldw: Int, oldh: Int) {
    super.onSizeChanged(w, h, oldw, oldh)
    drawingWidth = w
    drawingHeight = h
}
```

view의 화면크기가 변화 할때마자 사이즈 값을 파라미터로 return

```java
//곡선이 부드럽게 그려진다.
private val amplitudePaint = Paint(Paint.ANTI_ALIAS_FLAG).apply {
    color = context.getColor(R.color.purple_500)
    strokeWidth = LINE_WIDTH
    strokeCap = Paint.Cap.ROUND //양끝 처리
}
private var drawingAmplitudes: List<Int> = emptyList()

//음량의 크기에 따라 값을 가지고 그려줌
override fun onDraw(canvas: Canvas?) {
    super.onDraw(canvas)

    canvas ?: return

    val centerY = drawingHeight / 2f
    var offsetX = drawingWidth.toFloat()

    drawingAmplitudes
        .let {
            if(isReplaying){
                it.takeLast(replayingPosition)//뒤에서부터 값을 가져옴
            }else{
                it
            }
        }
        .forEach {
        val lineLength = it / MAX_AMPLITUDE * drawingHeight * 0.8F

        offsetX -= LINE_SPACE
        if (offsetX < 0) return@forEach

        //라인을 그리는 부분
        canvas.drawLine(
            offsetX,
            centerY - lineLength / 2F,
            offsetX,
            centerY + lineLength / 2F,
            amplitudePaint //위에서 선언했던 Paint객체를 상속받은 변수
        )
    }
}
```

별도의 쓰레드를 통해서 recorder의 값을 Main쓰레드부터 받아 list에 추가

##### SoundVisualizerView

```java
var onRequestCurrentAmplitude: (() -> Int)? = null

//별도 쓰레드로 계속 가져옴
private val visualizeRepeatAction: Runnable = object : Runnable {
    override fun run() {
        if(!isReplaying){
            //onRequestCurrentAmplitude을 통해서 mainActivity에서 받은 레코드 값을 받아온다.
            val currentAmplitude = onRequestCurrentAmplitude?.invoke() ?: 0
            //Amplitude, Draw
            drawingAmplitudes = listOf(currentAmplitude) + drawingAmplitudes
        }else{
            replayingPosition++
        }

        invalidate() //새로운 데이터를 받았을때 다시 draw해주게끔 해준다.

        handler?.postDelayed(this, ACTION_INTERVAL) //쓰레드 인터벌 시간 설정
    }
}
```

##### MainActivity

```java
private val soundVisualizerView: SoundVisualizerView by lazy {
    findViewById(R.id.soundVisualizerView)
}

private fun bindViews() {
    soundVisualizerView.onRequestCurrentAmplitude = {
        //위 함수를 실행하고 현재 recorder의 maxAmplitude값을 return
        recorder?.maxAmplitude?:0
    }
...
```

---

# Back

### Request Permissions

앱 실행 시 반드시 사용자에게 권한을 물어봐야하는 내용이다. 보통 사용자의 사생활에 영향을 줄 수 있는 영역에서 확인한다.

아래 이미지는 권한 요청 동작 흐름이다.

![image](https://user-images.githubusercontent.com/65350890/154689195-2ebf399e-46ed-4553-9dfb-cc6dfbe6a306.png)

● manifest에 사용할 권한을 추가해 준다.

```xml
<uses-permission android:name="android.permission.RECORD_AUDIO"/>
```

● Activity에서 권한 요청에대한 코드를 구성한다.

```java
//manifest에 설정한 권한을 배열로 담아 선언해 놓는다.
private val requiredPermission = arrayOf(Manifest.permission.RECORD_AUDIO)

override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)

    requestAudioPermission() //앱 실행시 권한 요청 실행
    initViews()
}

private fun requestAudioPermission(){
    /*
    권한을 요청하는 메서드로, manifest에서 설정한 권한과
    사용할 권한 코드를 인자로 넘겨 준다.
    */
    requestPermissions(requiredPermission, REQUEST_RECORD_AUDIO_PERMISSION)
}

//상수값으로 권한 코드 선언
companion object{
    private const val REQUEST_RECORD_AUDIO_PERMISSION = 201
}

//권한에 대한 결과를 받는 메서드를 오버라이드
override fun onRequestPermissionsResult(
    requestCode: Int,
    permissions: Array<out String>,
    grantResults: IntArray
) {
    super.onRequestPermissionsResult(requestCode, permissions, grantResults)

    /*
        요청 코드와 내가 설정한 요청 코드가 같고
        권한 결과와 디바이스 권한이 같은 경우를 변수에 담는다.
    */
    val audioRecordPermissionGranted =
        requestCode == REQUEST_RECORD_AUDIO_PERMISSION &&
            grantResults.firstOrNull() == PackageManager.PERMISSION_GRANTED
    //만약 권한이 false이면 앱 종료
    if(!audioRecordPermissionGranted){
        finish()
    }
}
```

### MediaRecorder

일반 오디오 및 동영상 포맷을 캡처하고 인코딩하는 지원 기능이 포함되어 있는 API
아래 이미지는 MediaRecorder의 상태 변화 흐름이다.

![image](https://user-images.githubusercontent.com/65350890/154498330-d411f907-bc1e-4f5f-8926-ea63e042df8c.png)

```java
private var recorder: MediaRecorder? = null

//용량이 클 수 있기 때문에 외부 캐쉬 디렉토리에 저장
private val recordingFilePath: String by lazy {
    "${externalCacheDir?.absolutePath}/recording.3gp"
}

private fun startRecording() {
    recorder = MediaRecorder().apply {
        setAudioSource(MediaRecorder.AudioSource.MIC)
        setOutputFormat(MediaRecorder.OutputFormat.THREE_GPP)
        setAudioEncoder(MediaRecorder.AudioEncoder.AMR_NB)
        setOutputFile(recordingFilePath)
        prepare()
    }
    recorder?.start()
    state = State.ON_RECORDING
}

//stop(), release()를 통해 동작을 멈추고 메모리 해제
private fun stopRecording() {
    recorder?.run {
        stop()
        release()
    }
    recorder = null
    state = State.AFTER_RECORDING
}
```

`setAudioSource()`마이크 기능을 사용 설정, `setOutputFormat()`,`setAudioEncoder()` 오디어 또는 동영상의 코덱 및 인코딩 방식에 대해 설정한다. 서로 호환되는 값으로 설정 해야한다. `setOutputFile()`은 변환한 파일을 저장할 위치를 잡아준다.

### MediaPlayer

MediaPlayer API를 사용하여 오디오 또는 동영상을 재생할 수 있다.

아래는 상태 흐름이다.
![image](https://user-images.githubusercontent.com/65350890/154500298-88a0d8c1-76c1-4aaf-9846-dbf73d387765.png)

```java
private var player: MediaPlayer? = null

private fun startPlaying() {
    player = MediaPlayer().apply {
        setDataSource(recordingFilePath)
        prepare()
    }
    player?.start()
    state = State.ON_PLAYING
}

private fun stopPlaying() {
    player?.release()
    player = null
    state = State.AFTER_RECORDING
}
```

`setDataSource()` 재생할 파일의 위치를 파라미터로 설정하여 소스 설정. `prepare()`를 통해 준비시켜놓고 `start()`해준다.

[전체 코드 보러 가기](https://github.com/byunginK/Andriod_Project/tree/main/chapter07)
