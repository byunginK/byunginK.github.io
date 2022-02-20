---
layout: post
title: "android basic 8"
date: 2022-02-20
categories: [Kotlin]
---

# UI

### webview

WebView 개체를 사용하면 활동 레이아웃의 일부로 웹 콘텐츠를 표시할 수 있다.

WebView는 앱을 위해 특별히 설계된 환경에 웹 페이지를 포함할 수 있는 고급 구성 옵션과 UI에 대한 제어를 강화해야 할 때 유용

```java
private val webView: WebView by lazy {
    findViewById(R.id.webView)
}

@SuppressLint("SetJavaScriptEnabled") //경고 무시 어노테이션
private fun initViews() {
    webView.apply {
        //우리 앱 웹뷰에 화면을 뿌려주도록 화면을 가져온다.
        webViewClient = WebViewClient()
        //화면내 자바스크립트가 default로 막아져 있으므로 동작을 위해 true
        settings.javaScriptEnabled = true
        //처음 로드 url 설정
        loadUrl(DEFAULT_URL)
        webChromeClient = WebChromeClient()
    }
}

companion object {
    private const val DEFAULT_URL = "http://www.google.com"
}
```

`WebViewClient()`와 `WebChromeClient()`은 inner 클래스로 생성한 웹뷰의 속성들을 상속받은 함수이다. 자세한 내용은 아래에서 설명
`imeOptions`

### swipeRefresh

아래로 스와이프하여 새로고침 UI 패턴을 구현합니다.

gradle에 아래와 같이 추가

```
implementation "androidx.swiperefreshlayout:swiperefreshlayout:1.1.0"
```

webView를 감싸도록 layout으로 설정

```xml
<androidx.swiperefreshlayout.widget.SwipeRefreshLayout
    android:id="@+id/refreshLayout"
    android:layout_width="0dp"
    android:layout_height="0dp"
    app:layout_constraintBottom_toBottomOf="parent"
    app:layout_constraintLeft_toLeftOf="parent"
    app:layout_constraintRight_toRightOf="parent"
    app:layout_constraintTop_toBottomOf="@id/toolbar">

    <WebView
        android:id="@+id/webView"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
</androidx.swiperefreshlayout.widget.SwipeRefreshLayout>
```

`MainActivity.kt`에서 선언해준 후 webView의 reload()함수에 걸어준다

```java
private fun bindView() {
    goHomeButton.setOnClickListener {
        webView.loadUrl(DEFAULT_URL)
    }

    addressBar.setOnEditorActionListener { textView, actionId, keyEvent ->
        if (actionId == EditorInfo.IME_ACTION_DONE) {
            val loadingUrl = textView.text.toString()
            if (URLUtil.isNetworkUrl(loadingUrl)) {
                webView.loadUrl(loadingUrl)
            } else {
                webView.loadUrl("http://$loadingUrl")
            }
        }

        return@setOnEditorActionListener false
    }

    goBackButton.setOnClickListener {
        webView.goBack()
    }

    goForwardButton.setOnClickListener {
        webView.goForward()
    }

    refreshLayout.setOnRefreshListener {
        webView.reload()
    }
}
```

또한 새로고침이 완료가 되면 스와이프 후 생기는 로드 ui를 사라지게 해줘야 한다. 아까 위에서 webView 속성에 설정할때 사용한 `WebViewClient`클래스에서 해당 설정을 오버라이드하여 진행한다.

```java
inner class WebViewClient : android.webkit.WebViewClient() {

    override fun onPageStarted(view: WebView?, url: String?, favicon: Bitmap?) {
        super.onPageStarted(view, url, favicon)
        progressBar.show()
    }

    override fun onPageFinished(view: WebView?, url: String?) {
        super.onPageFinished(view, url)
        refreshLayout.isRefreshing = false
        progressBar.hide()
        goBackButton.isEnabled = webView.canGoBack()
        goForwardButton.isEnabled = webView.canGoForward()
        addressBar.setText(url)
    }
}
```

class 내부에서 inner클래스로 생성하여 상속받아 webView를 initialize할때 넘겨준다.
`onPageStarted`와 `onPageFinished`는 각각 페이지가 시작, 로드가 끝났을때 아래 설명할 progress bar 설정과 뒤로가기, 앞으로가기 버튼 등 내용을 설정한다.

### progressBar

`ContentLoadingProgressBar`를 사용하여 layout에 그려줄 수 있으며,`MainActivity.kt`에서 언제 표시할지를 설정

```java
inner class WebChromeClient : android.webkit.WebChromeClient() {
    override fun onProgressChanged(view: WebView?, newProgress: Int) {
        super.onProgressChanged(view, newProgress)
        progressBar.progress = newProgress
    }
}
```

상속받아 webView 속성에 설정 값들을 넘겨준다.

## setOnEditorActionListener

EditView에서 imeOption을 통해 입력후 액션에대한 설정을 할 수 있다.

```xml
<EditText
    android:id="@+id/addressBar"
    android:layout_width="0dp"
    android:layout_height="32dp"
    android:background="@drawable/shape_address_bar"
    android:imeOptions="actionDone"
    android:importantForAutofill="no"
    android:inputType="textUri"
    android:paddingHorizontal="16dp"
    android:textSize="14sp"
    android:selectAllOnFocus="true"
    app:layout_constraintBottom_toBottomOf="parent"
    app:layout_constraintLeft_toRightOf="@id/goHomeButton"
    app:layout_constraintRight_toLeftOf="@id/goBackButton"
    app:layout_constraintTop_toTopOf="parent"
    tools:ignore="LabelFor" />
```

`android:imeOptions="actionDone"`을 설정하여 우측하단 끝 버튼을 누르면 실행 완료 옵션으로 설정 할 수 있다.
로직은 `MainActivity.kt`에서 설정

```java
    private fun bindView() {
        ...
        addressBar.setOnEditorActionListener { textView, actionId, keyEvent ->
            if (actionId == EditorInfo.IME_ACTION_DONE) {
                val loadingUrl = textView.text.toString()
                if (URLUtil.isNetworkUrl(loadingUrl)) {
                    webView.loadUrl(loadingUrl)
                } else {
                    webView.loadUrl("http://$loadingUrl")
                }
            }
            return@setOnEditorActionListener false
        }
        ...
    }
```

`EditorInfo.IME_ACTION_DONE`수정정보 코드와 실제 actionId값을 비교하여 로직을 실행하도록 설정하였다. 위에서 사용된 `URLUtil`의 `isNetworkUrl`은 url이 네트워크 url인지 boolean값 리턴한다.

# Back

### permission INTERNET

해당 앱에서 데이터, 인터넷을 사용할때 권한을 부여해야한다. manifest에 해당 권한을 추가해 준다.

```xml
<uses-permission android:name="android.permission.INTERNET"/>

<application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:usesCleartextTraffic="true"
        android:theme="@style/Theme.Chapper08">
        <activity
            android:name=".MainActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>
```

`android:usesCleartextTraffic="true"`라는 속성을 추가해줘야 하는데 이것은 앱이 일반 텍스트 HTTP와 같은 일반 텍스트 네트워크 트래픽을 사용하는지 여부를 나타낸다.

이 속성이 "false"로 설정되면 플랫폼 구성요소(예: HTTP 및 FTP 스택, DownloadManager, MediaPlayer)는 앱의 일반 텍스트 트래픽 사용 요청을 거부한다. _일반 텍스트 트래픽을 피하는 주요 이유는 기밀성, 진실성이 보장되지 않고 변조 방지가 불가능하기 때문_

### onBackPressed

안드로이드 '뒤로' 버튼을 눌렀을때 동작

```java
override fun onBackPressed() {
    if (webView.canGoBack()) {
        webView.goBack()
    } else {
        super.onBackPressed()
    }
}
```

[전체 코드 보러 가기](https://github.com/byunginK/Andriod_Project/tree/main/chapter07)
