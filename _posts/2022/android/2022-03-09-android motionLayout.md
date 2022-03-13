---
layout: post
title: "android motionLayout"
date: 2022-03-09
categories: [Kotlin]
---

# UI

### motionLayout

motionScene을 활용하여 화면을 구성할 수 있다. Fragment레이아웃에서 motionLayout을 연결을 해준다.

```java
private fun initMotionLayoutEvent(fragmentPlayerBinding: FragmentPlayerBinding){
    fragmentPlayerBinding.playerMotionLayout.setTransitionListener(object :MotionLayout.TransitionListener{
        override fun onTransitionStarted(
            motionLayout: MotionLayout?,
            startId: Int,
            endId: Int
        ) {

        }

        override fun onTransitionChange(
            motionLayout: MotionLayout?,
            startId: Int,
            endId: Int,
            progress: Float
        ) {
            binding?.let {
                (activity as MainActivity).also { mainActivity ->
                    mainActivity.findViewById<MotionLayout>(R.id.mainMotionLayout).progress = abs(progress)//절대값으로 프로그레스 값을 준다
                }
            }
        }

        override fun onTransitionCompleted(motionLayout: MotionLayout?, currentId: Int) {

        }

        override fun onTransitionTrigger(
            motionLayout: MotionLayout?,
            triggerId: Int,
            positive: Boolean,
            progress: Float
        ) {

        }

    })
}
```

motion레이아웃을 활용하게 되면 scene 파일이 생성되게 된다. 해당 xml에서 transition, constraionset에서 각 요소에 설정값을 주어 원하는 모션이 start, end가 되었을때 어떤 모양으로 설정될지 값을 줄 수 있다.

아래는 모션레이아웃을 활용했을때 커스텀설정을 통해 특정 화면부분에서만 제스처 했을때 동작되도록 설정하였다.

```java
//전체 화면이 스와프되는 문제를 위해 커스텀 생성
class CustomMotionLayout(context: Context, attributeSet: AttributeSet? = null):MotionLayout(context,attributeSet) {
    private var motionTouchStarted = false
    private val mainContainerView by lazy {
        findViewById<View>(R.id.mainContainerLayout)
    }
    private val hitRect = Rect()

    init {
        setTransitionListener(object : TransitionListener{
            override fun onTransitionStarted(
                motionLayout: MotionLayout?,
                startId: Int,
                endId: Int
            ) {

            }

            override fun onTransitionChange(
                motionLayout: MotionLayout?,
                startId: Int,
                endId: Int,
                progress: Float
            ) {

            }

            override fun onTransitionCompleted(motionLayout: MotionLayout?, currentId: Int) {
                motionTouchStarted = false
            }

            override fun onTransitionTrigger(
                motionLayout: MotionLayout?,
                triggerId: Int,
                positive: Boolean,
                progress: Float
            ) {

            }

        })
    }

    override fun onTouchEvent(event: MotionEvent): Boolean {
        when(event.actionMasked) {
            //ACTION_UP , ACTION_CANCEL 은 터치를 안하는 액션
            MotionEvent.ACTION_UP, MotionEvent.ACTION_CANCEL -> {
                motionTouchStarted = false
                return super.onTouchEvent(event) //원래값을 리턴 (사용 x)
            }
        }
        if(!motionTouchStarted){
            //getHitRect() = 상위 요소의 좌표에서 하위 요소의 적중 사각형(터치 가능한 영역)을 가져옵니다
            mainContainerView.getHitRect(hitRect)
            //rect.contains = Returns true if (x,y) is inside the rectangle
            motionTouchStarted = hitRect.contains(event.x.toInt(), event.y.toInt())
        }
        return super.onTouchEvent(event) && motionTouchStarted
    }

    private val gestureListener by lazy {
        object: GestureDetector.SimpleOnGestureListener(){
            override fun onScroll(
                e1: MotionEvent,
                e2: MotionEvent,
                distanceX: Float,
                distanceY: Float
            ): Boolean {
                mainContainerView.getHitRect(hitRect)
                return hitRect.contains(e1.x.toInt(),e1.y.toInt())
            }
        }
    }
    private val gestureDetector by lazy {
        /*
        Detects various gestures and events using the supplied MotionEvents.
        The OnGestureListener callback will notify users when a particular motion event has occurred.
        This class should only be used with MotionEvents reported via touch
        */
        GestureDetector(context, gestureListener)
    }
    override fun onInterceptTouchEvent(event: MotionEvent?): Boolean {
        return gestureDetector.onTouchEvent(event)
    }
}
```

### ExoPlayer

ExoPlayer는 Android 프레임워크에 속하지 않고 Android SDK에서 별도로 배포되는 오픈소스 프로젝트입니다. ExoPlayer의 표준 오디오 및 동영상 구성요소는 Android 4.1(API 레벨 16)에서 출시된 Android MediaCodec API를 기반으로 합니다.

아래는 exoPlayer를 init하는 코드이다.

```java
private fun initPlayer(fragmentPlayerBinding: FragmentPlayerBinding) {
    context?.let {
        player = SimpleExoPlayer.Builder(it).build()
    }

    fragmentPlayerBinding.playerView.player = player
    binding?.let {
        player?.addListener(object : Player.EventListener{
            override fun onIsPlayingChanged(isPlaying: Boolean) {
                super.onIsPlayingChanged(isPlaying)
                if(isPlaying){
                    it.bottomPlayerControlButton.setImageResource(R.drawable.ic_baseline_pause_24)
                }else{
                    it.bottomPlayerControlButton.setImageResource(R.drawable.ic_baseline_play_arrow_24)
                }
            }
        })
    }

}
```

실제 플레이하게끔 하는 코드

```java
fun play(url: String, title: String){

    context?.let {
        val dataSourceFactory = DefaultDataSourceFactory(it)
        val mediaSource = ProgressiveMediaSource.Factory(dataSourceFactory)
            .createMediaSource(MediaItem.fromUri(Uri.parse(url)))
        player?.setMediaSource(mediaSource)
        player?.prepare()
        player?.play()
    }

    binding?.let {
        it.playerMotionLayout.transitionToEnd() //모션 레이아웃에서 설정은 END의 화면으로 화면 전환
        it.bottomTitleTextView.text = title
    }
}
```

[전체 코드 보러 가기](https://github.com/byunginK/Andriod_Project/tree/main/chapter16)
