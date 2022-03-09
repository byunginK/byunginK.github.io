---
layout: post
title: "android fragment(bottom navigator)_floating button"
date: 2022-03-05
categories: [Kotlin]
---

# UI

### FrameLayout

FrameLayout은 단일 항목을 표시하기 위해 화면의 영역을 차단하도록 설계되었습니다. 일반적으로 FrameLayout은 자식 뷰가 서로 겹치지 않고 다양한 화면 크기로 확장 가능한 방식으로 자식 뷰를 구성하기 어려울 수 있으므로 단일 자식 뷰를 유지하는 데 사용해야 합니다.

### fragment (bottom navigator)

하단 탐색 모음을 사용하면 사용자가 탭 한 번으로 상위 수준 보기를 쉽게 탐색하고 전환할 수 있습니다.

메뉴 리소스 파일을 지정하여 막대 내용을 채울 수 있습니다.

**※ menu 리소스 (res 파일 자식으로 폴더 생성)**

```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:id="@+id/home"
        android:icon="@drawable/ic_baseline_home_24"
        android:title="홈"/>
    <item android:id="@+id/chatList"
        android:icon="@drawable/ic_baseline_chat_24"
        android:title="채팅"/>
    <item android:id="@+id/myPage"
        android:icon="@drawable/ic_baseline_person_pin_24"
        android:title="나의 정보"/>
</menu>
```

**※ Bottom Navigator**

```xml
<com.google.android.material.bottomnavigation.BottomNavigationView
    android:id="@+id/bottomNavigationView"
    android:layout_width="0dp"
    android:layout_height="wrap_content"
    app:itemIconTint="@drawable/selector_menu_color"
    app:itemTextColor="@color/black"
    app:layout_constraintBottom_toBottomOf="parent"
    app:layout_constraintEnd_toEndOf="parent"
    app:layout_constraintStart_toStartOf="parent"
    app:itemRippleColor="@null"
    app:menu="@menu/bottom_navigation_menu" />
```

### Fragment

Fragment는 앱 UI의 재사용 가능한 부분을 나타냅니다. 프래그먼트는 자체 레이아웃을 정의 및 관리하고 자체 수명 주기를 보유하며 자체 입력 이벤트를 처리할 수 있습니다. 프래그먼트는 독립적으로 존재할 수 없고 활동이나 다른 프래그먼트에서 호스팅되어야 합니다.

※ 모듈성

프래그먼트는 UI를 개별 청크로 분할할 수 있도록 하여 활동의 UI에 모듈성과 재사용성을 도입합니다.
작은 화면에서는 앱이 하단 탐색 메뉴와 선형 레이아웃 목록을 표시해야 합니다. Activity에서 이러한 모든 변형을 관리하는 작업은 어려울 수 있습니다. 콘텐츠에서 탐색 요소를 분리하면 이 프로세스를 더 쉽게 관리할 수 있습니다. 그러면 Activity은 올바른 탐색 UI를 표시하고 프래그먼트는 적절한 레이아웃으로 목록을 표시할 수 있다.

※ lifecycle
![image](https://user-images.githubusercontent.com/65350890/156917075-e251f962-86ad-4822-a3d1-5e9689cbb021.png)

**※ 연결**

MainActivity에 bottom Navigator와 fragment 화면들을 연결해준다.

```java
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val homeFragment = HomeFragment()
        val chatListFragment = ChatListFragment()
        val myPageFragment = MyPageFragment()

        val bottomNavigationView = findViewById<BottomNavigationView>(R.id.bottomNavigationView)

        //최초 화면을 홈으로 지정
        replaceFragment(homeFragment)

        //하단 네비게이터를 클릭 했을때 발생하는 이벤트 리스너너
        bottomNavigationView.setOnNavigationItemSelectedListener {
            when (it.itemId) {
                R.id.home -> replaceFragment(homeFragment)
                R.id.chatList -> replaceFragment(chatListFragment)
                R.id.myPage -> replaceFragment(myPageFragment)
            }
            true
        }
    }

    //전달 받은 fragment를 fragment컨테이너에 적용
    private fun replaceFragment(fragment: Fragment) {
        supportFragmentManager.beginTransaction()
            .apply {
                replace(R.id.fragmentContainer, fragment)
                commit()
            }
    }
}
```

아래는 homeFragment의 코드 일부

```java
class HomeFragment: Fragment(R.layout.fragment_home) {

    private var binding: FragmentHomeBinding? = null
    private lateinit var articleAdapter: ArticleAdapter
    private val auth: FirebaseAuth by lazy {
        Firebase.auth
    }
    private lateinit var articleDB: DatabaseReference
    private lateinit var userDB: DatabaseReference
    private val articleList = mutableListOf<ArticleModel>()

    ...

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        val fragmentHomeBinding = FragmentHomeBinding.bind(view)
        binding = fragmentHomeBinding
        //layout에 fragment_home.xml 맵핑 (gradel 앱단위에서 viewBinding enabled =true 설정)
        ...
    }

    override fun onDestroy() {
        super.onDestroy()

        articleDB.removeEventListener(listener)
    }

    @SuppressLint("NotifyDataSetChanged")
    override fun onResume() {
        super.onResume()
        //뷰를 다시그럼
        articleAdapter.notifyDataSetChanged()
    }
}
```

### Floating Button

```xml
<com.google.android.material.floatingactionbutton.FloatingActionButton
    android:id="@+id/addFloatingButton"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_margin="16dp"
    android:backgroundTint="@color/orange"
    android:src="@drawable/ic_baseline_add_24"
    app:layout_constraintBottom_toBottomOf="parent"
    app:layout_constraintEnd_toEndOf="parent"
    app:tint="@color/white" />
```

클릭 이벤트 리스너를 사용하여 버튼 클릭시 로직 구현

```java
fragmentHomeBinding.addFloatingButton.setOnClickListener {
    //context의 safe call을 사용하여 intent에 context에 넣어준다
    context?.let {
        if(auth.currentUser != null){
            val intent = Intent(it,AddArticleActivity::class.java)
            startActivity(intent)
        }else{
            Snackbar.make(view,"로그인 후 사용해주세요", Snackbar.LENGTH_LONG).show()
        }

    }

}
```

[전체 코드 보러 가기](https://github.com/byunginK/Andriod_Project/tree/main/chapter14)
