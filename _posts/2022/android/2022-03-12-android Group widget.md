---
layout: post
title: "android view group & data model"
date: 2022-03-12
categories: [Kotlin]
---

# UI

인자를 받아서 각각 생성자의 설정 값을 줄 수 있게 생성자를 함수화하여 사용 할 수 있다.

[PlayerFragment.kt]

```java
    companion object {
        //인자를 받아서 각각 생성자의 설정값을 삽입할 수있게 함수화 하여 사용
        fun newInstance(): PlayerFragment {
            return PlayerFragment()
        }
    }
```

[MainActivity.kt]

```java
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        supportFragmentManager.beginTransaction()
            .replace(R.id.fragmentContainer, PlayerFragment.newInstance())
            .commit()
    }
}
```

`newInstance()`를 통해 fragmentLayout로 사용

### Group

`ConstraintLayout`에서 여러개의 요소들을 그룹화하여 한번에 설정을 할 수 있도록 한다.
예를 들어 음악 플레이어의 경우 재생 목록과 재생화면이 다른데 visible을 한번에 적용 할 수 있다.

[fragment_player.xml]

```xml
<androidx.constraintlayout.widget.ConstraintLayout>
    <androidx.constraintlayout.widget.Group
        android:id="@+id/playerViewGroup"
        app:constraint_referenced_ids="trackTextView, artistTextView, coverImageCardView,
        bottomBackgroundView, playerSeekBar, playTimeTextView, totalTimeTextView"
        android:visibility="gone"
        tools:visibility="visible"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"/>

    <androidx.constraintlayout.widget.Group
        android:id="@+id/playerListViewGroup"
        app:constraint_referenced_ids="titleTextView,playListRecyclerView,playlistSeekBar"
        android:visibility="visible"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"/>

    ...

</androidx.constraintlayout.widget.ConstraintLayout>
```

그리고 버튼 이벤트를 통해 그룹의 설정값을 바꿀 수 있다.

[PlayerFragment.kt]

```java
private fun initPlayListButton(fragmentPlayerBinding: FragmentPlayerBinding) {
    //리스트 버튼 그룹
    fragmentPlayerBinding.playlistImageView.setOnClickListener {
        //만약 서버에서 데이터가 다 불러오지 않은 상태 일때 예외처리 코드 필요
        if (model.currentPosition == -1) return@setOnClickListener
        fragmentPlayerBinding.playerViewGroup.isVisible = model.isWatchingPlayListView
        fragmentPlayerBinding.playerListViewGroup.isVisible = model.isWatchingPlayListView.not()
        model.isWatchingPlayListView = !model.isWatchingPlayListView
    }
}
```

# Back

### Entity와 Model의 mapping

[MusicEntity]
API의 데이터를 직접 받아오는 Entity

```java
data class MusicEntity(
    @SerializedName("track") val track: String,
    @SerializedName("streamUrl") val streamUrl: String,
    @SerializedName("artist") val artist: String,
    @SerializedName("coverUrl") val coverUrl: String,
)
```

[MusicDto]
Entity를 List로 collect 해준다.

```java
data class MusicDto(
    val musics: List<MusicEntity>
)
```

[MusicModel]
받아온 MusicEntity에 id와 현재 상태값을 추가하여 Model로 생성

```java
data class MusicModel(
    val id: Long,
    val track: String,
    val streamUrl: String,
    val artist: String,
    val coverUrl: String,
    val isPlaying: Boolean = false
)
```

[MusicModelMapper]
확장함수를 활용하여 dto 또는 Entity를 Model의 원하는 값으로 설정하는 로직

```java
fun MusicEntity.mapper(id: Long): MusicModel =
    MusicModel(
        id = id,
        streamUrl = streamUrl,
        coverUrl = coverUrl,
        track = track,
        artist = artist
    )

fun MusicDto.mapper():PlayerModel =
    PlayerModel(
        playMusicList = musics.mapIndexed {index, musicEntity ->
            musicEntity.mapper(index.toLong()) //musicModel의 리스트로 반환
        }
    )
```

[PlayerModel]
추가적인 상태값과 MusicModel등을 받는 모델 생성

```java
data class PlayerModel(
    private val playMusicList: List<MusicModel> = emptyList(),
    var currentPosition: Int = -1,
    var isWatchingPlayListView: Boolean = true
) {
    //만약 그냥 기존 모델에서 값만 바꾸게 되면 참조 주소가 동일하게되고 그러면 adapter diffutil에서 old와 new가 다른것을 인지하지 못한다
    fun getAdapterModels(): List<MusicModel> {
        return playMusicList.mapIndexed { index, musicModel ->
            val newItem = musicModel.copy( //값은 그대로 가져오면서 클래스르 새로 만든다.
                isPlaying = index == currentPosition
            )
            newItem
        }
    }

    fun updateCurrentPosition(musicModel: MusicModel) {
        currentPosition = playMusicList.indexOf(musicModel)
    }

    fun nextMusic(): MusicModel? {
        if (playMusicList.isEmpty()) return null
        currentPosition =
            if ((currentPosition + 1) == playMusicList.size) 0 else currentPosition + 1
        return playMusicList[currentPosition]
    }

    fun prevMusic(): MusicModel? {
        if (playMusicList.isEmpty()) return null
        currentPosition =
            if ((currentPosition - 1) < 0) playMusicList.lastIndex else currentPosition - 1
        return playMusicList[currentPosition]
    }

    fun currentMusicModel(): MusicModel? {
        if (playMusicList.isEmpty()) return null
        return playMusicList[currentPosition]
    }
}
```

[PlayerFragment]
API에서 데이터를 받아오는 로직

```java
private fun getVideoListFromServer() {
    val retrofit = Retrofit.Builder()
        .baseUrl("https://run.mocky.io")
        .addConverterFactory(GsonConverterFactory.create())
        .build()

    retrofit.create(MusicService::class.java)
        .also {
            it.listMusics()
                .enqueue(object : Callback<MusicDto> {
                    override fun onResponse(
                        call: Call<MusicDto>,
                        response: Response<MusicDto>
                    ) {
                        response.body()?.let { musicDto ->
                        //mapper를 통해 playerModel에 dto(List)의 index를 id로 설정 후 반환
                            model = musicDto.mapper()

                            setMusicList(model.getAdapterModels())
                            playListAdapter.submitList(model.getAdapterModels())
                        }
                    }

                    override fun onFailure(call: Call<MusicDto>, t: Throwable) {

                    }

                })
        }
}
```

### seekBar 업데이트

Runnable 와 postDelayed를 활용하여 일정시간마다 추가 쓰레드로 로직을 실행하도록 구현

```java
private val updateSeekRunnable = Runnable {
    updateSeek()
}

//postDelayed를 사용하여 무한 루프를 사용, 탈출 조건은 playbackState
private fun updateSeek() {
    val player = this.player ?: return
    val duration = if (player.duration >= 0) player.duration else 0
    val position = player.currentPosition

    updateSeekUi(duration, position)
    val state = player.playbackState

    view?.removeCallbacks(updateSeekRunnable)
    if (state != Player.STATE_IDLE && state != Player.STATE_ENDED) {
        view?.postDelayed(updateSeekRunnable, 1000)
    }
}
```

### seekTo

MediaPlayer의 메소드로 용도는 아래와 같다.
미디어를 지정된 시간 위치로 이동.
공식 문서 url : (https://developer.android.com/reference/android/media/MediaPlayer#seekTo(long,%20int))

```java
private fun playMusic(musicModel: MusicModel) {
    model.updateCurrentPosition(musicModel)
    //미디어의 현재 position 과 mode
    player?.seekTo(model.currentPosition, 0)
    player?.play()
}

private fun initSeekBar(fragmentPlayerBinding: FragmentPlayerBinding) {
    fragmentPlayerBinding.playerSeekBar.setOnSeekBarChangeListener(object : SeekBar.OnSeekBarChangeListener{
        override fun onProgressChanged(seekBar: SeekBar?, p1: Int, p2: Boolean) {

        }

        override fun onStartTrackingTouch(p0: SeekBar?) {
        }

//완전히 터치가 끝났을 상황을 의미
        override fun onStopTrackingTouch(seekBar: SeekBar) {
            player?.seekTo((seekBar.progress*1000).toLong())
        }

    })

    fragmentPlayerBinding.playlistSeekBar.setOnTouchListener { view, motionEvent ->
        false
    }
}
```

[전체 코드 보러 가기](https://github.com/byunginK/Andriod_Project/tree/main/chapter17)
