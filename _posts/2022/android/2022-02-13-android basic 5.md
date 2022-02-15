---
layout: post
title: "android basic 5"
date: 2022-02-13
categories: [Kotlin]
---

# UI

### layout_constraintDimensionRatio

요소를 비율에 맞게 분할 배치하도록 한다.

```xml
app:layout_constraintDimensionRatio="H,3:1"
```

_수평으로 3개를 1줄_

### 가로 모드

`Manifest.xml`에 activity에 속성을 추가

```xml
<activity android:name=".PhotoFrameActivity"
    android:screenOrientation="landscape"/>
```

## View animation

타이머를 설정하여 시간이 지날때마다 이미지가 바뀌는 기능에서 `imageView`에 에니메이션을 추가하여 이미지가 변경될때 투명하게 사라지도록 설정

```java
private fun startTimer(){
    timer = kotlin.concurrent.timer(period = 5000){
        runOnUiThread {
            val current = currentPosition
            val next = if(photoList.size <= currentPosition +1) 0 else currentPosition + 1

            backgroundPhotoImageView.setImageURI(photoList[current])

            photoImageView.alpha = 0f //투명도 초기 0
            photoImageView.setImageURI(photoList[next])
            photoImageView.animate()
                .alpha(1.0f) //1로 바꾸어 보이게끔 설정
                .setDuration(1000)
                .start()
            currentPosition = next
        }
    }
}
```

---

# Back

### Android Permission

사용자의 민감한 데이터에 접근을 하기 위해서는 권한이 필요하며 권한 동의에 대해 허용을 받아 낼 수 있다.

```java
private fun initAddPhotoButton() {
    addPhotoButton.setOnClickListener {
        when
        {
            ContextCompat.checkSelfPermission(this, android.Manifest.permission.READ_EXTERNAL_STORAGE) == PackageManager.PERMISSION_GRANTED -> {
                navigatePhotos()
            }
            shouldShowRequestPermissionRationale(
                android.Manifest.permission.READ_EXTERNAL_STORAGE
            ) -> {
                showPermissionContextPopup()
            }
            else -> {
                requestPermissions(
                    arrayOf(android.Manifest.permission.READ_EXTERNAL_STORAGE),
                    1000
                )
            }
        }
    }
}
```

`checkSelfPermission()`를 통해 갤러리 저장공간의 읽기 권한을 확인한다. 권한이 있다면 `navigatePhotos()` 을 실행한다. 별도의 팝업을 띄우고 싶다면 `shouldShowRequestPermissionRationale()`를 통해 팝업을 띄우고 권한을 받도록 설정 할 수 있다.

`requestPermissions()`에서 arrayOf를 통해 권한들을 넘길 수있다. 권한에는 `requestCode`를 부여하게 되고 해당 코드를 통해 실행되는 로직을 구현 할 수 있다.

**권한결과를 받고나서 실행되는 로직**

```java
override fun onRequestPermissionsResult(
    requestCode: Int,
    permissions: Array<out String>,
    grantResults: IntArray
) {
    super.onRequestPermissionsResult(requestCode, permissions, grantResults)
    //위의 권한에서 설정한 requestcode를 통해 분절
    when (requestCode) {
        1000 -> {
            if(grantResults.isNotEmpty() && grantResults[0] == PackageManager.PERMISSION_GRANTED){
                navigatePhotos()
            }else{
                Toast.makeText(this,"권한을 거부하였습니다.",Toast.LENGTH_SHORT).show()
            }
        }
        else -> {

        }
    }
}
```

**※** 또한 권한을 사용하기 위해서는 반드시 `Manifest.xml`에 선언을 해주어야 한다.

```xml
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
```

### SAF(Storage Access Framework)

문서 및 이미지 등 각종 파일을 탐색하고 저장하는 작업을 간편하게 하려고 도입

```java
//SFA를 사용하여 가져옴
private fun navigatePhotos() {
    val intent = Intent(Intent.ACTION_GET_CONTENT)
    intent.type = "image/*" //이미지 파일만 열림
    startActivityForResult(intent,2000)
}
```

위에서 선택된 이미지를 intent로 넘겨서 `onActivityResult()`처리하도록 아래와 같이 구성.
`requestCode` 에 따라 처리되는 로직을 구분할 수 있다. `resultCode`는 처리가 되었는지 상태를 확인 할 수 있다.

```java
override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
    super.onActivityResult(requestCode, resultCode, data)
    if(resultCode != Activity.RESULT_OK){
        return
    }

    when(requestCode){
        2000 -> {
            val selectedImageUri: Uri? = data?.data
            if(selectedImageUri != null){

                if(imageUriList.size == 6){
                    Toast.makeText(this, "이미 사진이 꽉 찼습니다.",Toast.LENGTH_SHORT).show()
                    return
                }

                imageUriList.add(selectedImageUri)
                imageViewList[imageUriList.size-1].setImageURI(selectedImageUri)
            }else{
                Toast.makeText(this, "사진을 가져오지 못했습니다.",Toast.LENGTH_SHORT).show()
            }
        }
        else -> {
            Toast.makeText(this, "사진을 가져오지 못했습니다.",Toast.LENGTH_SHORT).show()
        }
    }
}
```

위에서 받은 이미지를 다른 Activity로 넘길때는 아래와 같이 넘겼다.

```java
private fun initStartPhotoFrameModeButton() {
    startPhotoFrameModeButton.setOnClickListener {
        val intent = Intent(this, PhotoFrameActivity::class.java)
        imageUriList.forEachIndexed { index, uri ->
            intent.putExtra("photo$index",uri.toString())
        }
        intent.putExtra("photoListSize",imageUriList.size)
        startActivity(intent)
    }
}
```

_PhotoFrameActivity에서 받을때_

```java
private fun getPhotoUriFromIntent(){
    val size = intent.getIntExtra("photoListSize",0)
    for(i in 0..size){
        intent.getStringExtra("photo$i")?.let {
            photoList.add(Uri.parse(it))
        }
    }
}
```

## Activity LifeCycle

앱이 사용될때, 죽어있을때, 다른 앱으로 전환 되었을때, 등 수명 주기에 대한 콜백을 잘 구성해야 안정적으로 앱이 구동된다.

[활동 수명 주기에 관한 이해](https://developer.android.com/guide/components/activities/activity-lifecycle)

![image](https://user-images.githubusercontent.com/65350890/153749001-9213d201-a568-4ca6-aaad-bba394974b56.png)

따라서 아래 앱에서 다음과 같이 수명 주기 관리를 할 수 있다.

```java
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)

    setContentView(R.layout.activity_photoframe)
    getPhotoUriFromIntent()
}
override fun onStop() {
    super.onStop()
    timer?.cancel()
}

override fun onStart() {
    super.onStart()
    startTimer()
}

override fun onDestroy() {
    super.onDestroy()
    timer?.cancel()
}
```
