---
layout: post
title: "android bottom Sheet Behavior"
date: 2022-03-06
categories: [Kotlin]
---

# UI

### CoordinatorLayout

CoordinatorLayout은 두 가지 주요 사용 사례를 위한 것입니다.

1. 최상위 애플리케이션 데코 또는 크롬 레이아웃
2. 하나 이상의 하위 뷰와의 특정 상호 작용을 위한 컨테이너

CoordinatorLayout의 자식 보기에 대해 지정 Behaviors하면 단일 부모 내에서 다양한 상호 작용을 제공할 수 있으며 이러한 보기도 서로 상호 작용할 수 있습니다.

Behaviors 는 슬라이딩 서랍 및 패널에서 이동 및 애니메이션 시 다른 요소에 달라붙는 스와이프 해제 가능한 요소 및 버튼에 이르기까지 다양한 상호 작용 및 추가 레이아웃 수정을 구현하는 데 사용할 수 있습니다.

```xml
<androidx.coordinatorlayout.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <com.naver.maps.map.MapView
        android:id="@+id/mapView"
        android:layout_marginBottom="80dp"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />


    <androidx.viewpager2.widget.ViewPager2
        android:id="@+id/houseViewPager"
        android:layout_width="match_parent"
        android:layout_height="100dp"
        android:layout_gravity="bottom"
        android:layout_marginBottom="120dp"
        android:orientation="horizontal" />

    <com.naver.maps.map.widget.LocationButtonView
        android:id="@+id/currentLocationButton"
        android:layout_gravity="top|start"
        android:layout_margin="12dp"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"/>
    <include
        android:id="@+id/include"
        layout="@layout/bottom_sheet" />

</androidx.coordinatorlayout.widget.CoordinatorLayout>
```

위 코드에서 include하여 behavior을 적용한 뷰를 추가한다.
(맨 마지막에 선언하여 위의 viewpager 위에서 작동하도록 한다.)

### bottom Sheet Behavior (bottom Sheet Dialog)

CoordinatorLayout에서 behavior를 주어 하단에서 슬라이드하여 올라오고 내려오는 행위를 설정 할 수 있다.
(단, bottom Sheet Dialog가 별도로 존재하여 이것을 사용하는것이 더 좋다고 함)

```xml
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@drawable/top_radius_white_background"
    app:behavior_peekHeight="100dp"
    app:layout_behavior="com.google.android.material.bottomsheet.BottomSheetBehavior">

    <View
        android:layout_width="30dp"
        android:layout_height="3dp"
        android:layout_marginTop="12dp"
        android:background="#cccccc"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <TextView
        android:id="@+id/bottomSheetTitleTextView"
        android:layout_width="0dp"
        android:layout_height="100dp"
        android:gravity="center"
        android:text="여러개의 숙소"
        android:textColor="@color/black"
        android:textSize="15sp"
        android:textStyle="bold"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <View
        android:id="@+id/lineView"
        android:layout_width="0dp"
        android:layout_height="1dp"
        android:background="#cccccc"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@id/bottomSheetTitleTextView" />

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/recyclerView"
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@id/lineView" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

ConstraintLayout 속성으로 `layout_behavior`와 `behavior_peekHeight`을 추가하여 구성

## 네이버 지도 API

앱 단위 gradle에 추가

```xml
implementation 'com.naver.maps:map-sdk:3.14.0'
```

디바이스 현재위치 권한을 위해 아래 의존성도 추가해준다.

```xml
implementation 'com.google.android.gms:play-services-location:19.0.1'
```

`setting.gradle`에 maven에 naver를 추가해준다.

```xml
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
        maven {
            url 'https://naver.jfrog.io/artifactory/maven/'
        }
    }
}
```

그리고 `gradle.properties`에 `android.enableJetifier=true`를 추가해준다.

MainActivity에 OnMapReadyCallback를 상속하고 지도를 생성한다.

```java
class MainActivity : AppCompatActivity(), OnMapReadyCallback, Overlay.OnClickListener {

    private val mapView: MapView by lazy {
        findViewById(R.id.mapView)
    }
    private lateinit var naverMap: NaverMap
    private lateinit var locationSource: FusedLocationSource
    private val bottomSheetTitleTextView: TextView by lazy {
        findViewById(R.id.bottomSheetTitleTextView)
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        mapView.onCreate(savedInstanceState)

        mapView.getMapAsync(this)

        viewPager.adapter = viewPagerAdapter
        recyclerView.adapter = recyclerAdapter
        recyclerView.layoutManager = LinearLayoutManager(this)
    }
    //OnMapReadyCallback 상속을 통해 구현체를 가지고와 위 getMapAsync에 객체로 넣을 수 있게 된다.
    //naver맵객체에 설정을 할 수 있다.
    override fun onMapReady(map: NaverMap) {
        naverMap = map
        naverMap.maxZoom = 18.0
        naverMap.minZoom = 10.0

        //위도 경도를 넣어 카메라 이동
        val cameraUpdate = CameraUpdate.scrollTo(LatLng(37.28845055747363, 127.05167605402836))
        naverMap.moveCamera(cameraUpdate)

        val uiSetting = naverMap.uiSettings
        uiSetting.isLocationButtonEnabled = false //현재 위치 버튼
        currentLocationButton.map = naverMap


        locationSource = FusedLocationSource(this@MainActivity,LOCATION_PERMISSION_REQUEST_CODE)
        naverMap.locationSource = locationSource


        getHouseListFromAPI()
    }
    //구글의 라이브러리 com.google.android.gms:play-services-location 이용하여 권한을 받아온다
    override fun onRequestPermissionsResult(
        requestCode: Int,
        permissions: Array<out String>,
        grantResults: IntArray
    ) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults)
        if(requestCode != LOCATION_PERMISSION_REQUEST_CODE){
            return
        }
        if(locationSource.onRequestPermissionsResult(requestCode, permissions, grantResults)){
            if(!locationSource.isActivated){
                naverMap.locationTrackingMode = LocationTrackingMode.None
            }
            return
        }
    }
    ...
}
```

[전체 코드 보러 가기](https://github.com/byunginK/Andriod_Project/tree/main/chapter16)
