---
layout: post
title: "android retrofit_glide"
date: 2022-02-28
categories: [Kotlin]
---

# Back

### Retrofit

API를 호출할때 사용하는 라이브러리다.( http 통신)

사용을 하기 위해서는 Gradle에 추가를 해주어야 한다.

```
implementation 'com.squareup.retrofit2:retrofit:2.9.0'
implementation 'com.squareup.retrofit2:converter-gson:2.9.0'
```

`converter-gson`은 응답결과를 json형태로 변환해주는 라이브러리다.

네트워크 통신을 위해 manifest에 권한을 부여하고 https로 통신을 권장하지만 그럴수 없다면 http로 통신을 해야하기 때문에 트레픽 설정도 해준다.

```xml
<uses-permission android:name="android.permission.INTERNET"/>

<!-- <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true" -->
        android:usesCleartextTraffic="true"
        <!-- android:theme="@style/Theme.Chapter12">
        <activity
            android:name=".MainActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        <activity android:name=".DetailActivity"/>
    </application> -->

```

서비스할 인터페이스를 구성 한다. (메서드 정의)

```java
import retrofit2.Call
import retrofit2.http.GET
import retrofit2.http.Query

interface BookService {
    @GET("/api/search.api?&output=json")
    fun getBooksByName(
        @Query("key") apiKey: String,
        @Query("query")keyword: String
    ): Call<SearchBookDto>

    @GET("/api/bestSeller.api?&categoryId=100&output=json")
    fun getBestSellerBooks(
        @Query("key") apiKey: String
    ):Call<BestSellerDto>
}
```

응답 받은 객체를 담기위해 data class를 선언한다.

```java
import android.os.Parcelable
import com.google.gson.annotations.SerializedName
import kotlinx.parcelize.Parcelize

@Parcelize
data class Book(
    //@SerializedName = 실제 데이터의 명칭과 동일하게 맵핑
    @SerializedName("itemId") val id: String,
    @SerializedName("title") val title: String,
    @SerializedName("description") val description: String,
    @SerializedName("coverSmallUrl") val coverSmallUrl: String,

): Parcelable //직렬화를 가능하도록 설정
```

직렬화를 하여 응답받은 데이터를 바로 변환할 수 있도록 해준다.

아래와 같이 레프토핏을 선언하고 인터페이스에 구성한 서비스를 생성하여 응답을 받아온다.

```java
private lateinit var bookService: BookService
val retrofit = Retrofit.Builder()
    .baseUrl("https://book.interpark.com")
        //바로 json형태로 변환하도록 설정
    .addConverterFactory(GsonConverterFactory.create())
    .build()

bookService = retrofit.create(BookService::class.java)

//아래 기능을 사용하기 위해서는 internet 권한을 줘야함
bookService.getBestSellerBooks(getString(R.string.interparkAPIKey))
    .enqueue(object: Callback<BestSellerDto>{
        override fun onResponse(
            call: Call<BestSellerDto>,
            response: Response<BestSellerDto>
        ) {
            //성공처리
            if(response.isSuccessful.not()){
                Log.e(TAG,"NOT SUCCESS!!")
                return
            }
            response.body()?.let {
                //currentList에 리스트를 넘겨준다.
                adapter.submitList(it.books)
            }
        }
        override fun onFailure(call: Call<BestSellerDto>, t: Throwable) {
            //실패처리
            Log.e(TAG, t.toString() )
        }
    })
```

### glide

이미지 로딩을 간편하게 해주는 API이다.

gradle에 의존성 주입 해준다.

```xml
implementation 'com.github.bumptech.glide:glide:4.12.0'
```

이후 아래와 같이 사용하면된다.

```java
//이미지 라이브러리 사용하여 이미지를 binding
Glide
    .with(binding.coverImageView.context) //context
    .load(bookModel.coverSmallUrl) //이미지 경로
    .into(binding.coverImageView) // imageView
```

[전체 코드 보러 가기](https://github.com/byunginK/Andriod_Project/tree/main/chapter12)
