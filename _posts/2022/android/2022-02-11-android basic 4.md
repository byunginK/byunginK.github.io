---
layout: post
title: "android basic 4"
date: 2022-02-11
categories: [Kotlin]
---

## UI

### TableLayout

일정한 칸 및 네모틀의 레이아웃을 사용할 때 주로 사용된다. `TableRow`를 통해 내부 칸을 나눌 수 있으며 row안에 내용을 넣어 표현할 수 있다.

### android:shrinkColumns

Row에 표현된 요소들이 범위를 벗어날때 해당 속성을 이용하여 줄어들게 설정 후 한 칸에 다 표현될 수 있도록 한다.
(\*는 모든 요소에 적용을 의미)

```xml
<TableLayout
    android:id="@+id/keypadTableLayout"
    android:layout_width="0dp"
    android:layout_height="0dp"
    android:paddingStart="15dp"
    android:paddingTop="21dp"
    android:paddingEnd="15dp"
    android:paddingBottom="21dp"
    android:shrinkColumns="*"
    app:layout_constraintBottom_toBottomOf="parent"
    app:layout_constraintEnd_toEndOf="parent"
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintTop_toBottomOf="@id/topLayout"
    app:layout_constraintVertical_weight="1.5">
```

### android:onClick

버튼 클릭시 해당 메소드를 Activity파일에서 매핑 해줄 수 있도록 한다.
Activity파일에서 함수로 바로 로직 구현이 가능하다.

```xml
android:onClick="clearButtonClicked"
```

```java
fun clearButtonClicked(v: View) {
    expressionTextView.text = ""
    resultTextView.text = ""
    isOperator = false
    hasOperator = false
}
```

### ripple

버튼을 drawable하여 설정하였다. 버튼 클릭시 색깔이 바뀌는 모양

```xml
<ripple xmlns:android="http://schemas.android.com/apk/res/android"
    android:color="@color/buttonPressGray">

    <item android:id="@android:id/background">
        <shape android:shape="rectangle">
            <solid android:color="@color/buttonGray" />
            <corners android:radius="100dp" />
            <stroke
                android:width="1dp"
                android:color="@color/buttonPressGray" />
        </shape>
    </item>

</ripple>
```

main.xml에 버튼 클릭시 애니메이션을 리플로 하기 때문에 다른 상태는 null값 처리

```xml
android:stateListAnimator="@null"
```

### ScrollView

해당 뷰 안에 요소들이 지정 범위를 벗어나게되면 스크룰이 생성되는 뷰
아래 예시는 스크롤 뷰 내부에 LiearLayout을 사용하여 세로로 아이템들이 그려지는 형식

```xml
<ScrollView
    android:layout_margin="10dp"
    android:layout_width="0dp"
    android:layout_height="0dp"
    app:layout_constraintEnd_toEndOf="parent"
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintTop_toBottomOf="@id/closeButton"
    app:layout_constraintBottom_toTopOf="@id/historyClearButton">

    <LinearLayout
        android:id="@+id/historyLinearLayout"
        android:orientation="vertical"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>
</ScrollView>
```

### SpannableStringBuilder

내용과 마크업을 모두 변경할 수 있는 텍스트의 클래스
아래 예시는 연산자의 색을 변경하기위해 사용하였다.

```java
val ssb = SpannableStringBuilder(expressionTextView.text)
ssb.setSpan(
    ForegroundColorSpan(getColor((R.color.green))),
    expressionTextView.text.length -1,
    expressionTextView.text.length,
    Spannable.SPAN_EXCLUSIVE_EXCLUSIVE)
```

## Back

### 확장함수

기존 클래스에 함수를 확장하여 사용 할 수 있다. 아래 예시처럼 String으로 숫자값을 받아 해당 값이 숫자인지 판별하는 확장함수를 구현 할 수 있다.

```java
//확장함수 구현
fun String.isNumber(): Boolean{
    return try {
        this.toBigInteger()
        true
    } catch (e: NumberFormatException){
        false
    }

}


fun resultButtonClicked(v: View) {
    val expressionText = expressionTextView.text.split(" ")

    if(expressionTextView.text.isEmpty() || expressionText.size == 1){
        return
    }
    if(expressionText.size != 3 && hasOperator){
        Toast.makeText(this,"아직 완성되지 않은 수식입니다.",Toast.LENGTH_SHORT).show()
        return
    }
    if(expressionText[0].isNumber().not() || expressionText[2].isNumber().not()){
        Toast.makeText(this,"아직 완성되지 않은 수식입니다.",Toast.LENGTH_SHORT).show()
        return
    }
    val expressionTxt = expressionTextView.text.toString()
    val resultText = calculateExpression()


    Thread(Runnable {
        db.historyDao().insertHistory(History(null,expressionTxt,resultText))
    }).start()

    resultTextView.text = ""
    expressionTextView.text = resultText

    isOperator = false
    hasOperator = false
}
```

### 로컬 데이터베이스 사용

Room을 사용하여 로컬 데이터베이스에 데이터를 저장 및 조회하여 사용 할 수 있다.

```
상당한 양의 구조화된 데이터를 처리하는 앱은 데이터를 로컬에 유지하여 매우 큰 이익을 얻을 수 있습니다. 가장 일반적인 사용 사례는 기기가 네트워크에 액세스할 수 없을 때도 사용자가 오프라인 상태로 계속 콘텐츠를 탐색할 수 있도록 관련 데이터를 캐시하는 것입니다.

Room 지속성 라이브러리는 SQLite를 완벽히 활용하면서 원활한 데이터베이스 액세스가 가능하도록 SQLite에 추상화 계층을 제공합니다. 특히 Room을 사용하면 다음과 같은 이점이 있습니다.

- SQL 쿼리의 컴파일 시간 확인
- 반복적이고 오류가 발생하기 쉬운 상용구 코드를 최소화하는 편의 주석
- 간소화된 데이터베이스 이전 경로
이러한 점을 고려할 때 SQLite API를 직접 사용하는 대신 Room을 사용하는 것이 좋습니다.
```

#### 1. gradle 종속 추가

**build.gradle(Moduler:chapter04.app)**

아래 항목들을 추가후 빌드

```gradle
plugins {
    id 'kotlin-kapt'
}

dependencies {

    implementation "androidx.room:room-runtime:2.4.1"
    kapt "androidx.room:room-compiler:2.4.1"
}
```

#### 2. model 생성

data class를 사용하여 저장할 값의 객체를 생성하도록 한다. 해당 모델을 통해 테이블 및 컬럼을 설정한다.

```java

import androidx.room.ColumnInfo
import androidx.room.Entity
import androidx.room.PrimaryKey

@Entity
data class History(
    @PrimaryKey val uid: Int?,
    @ColumnInfo(name = "expression") val expression: String?,
    @ColumnInfo(name = "result") val result: String?
)
```

@PrivmarKey는 고유한 값을 생성한다.

#### 3. Dao 생성

인터페이스를 생성하여 실행할 함수(메소드)를 설계한다.

```java
import androidx.room.Dao
import androidx.room.Delete
import androidx.room.Insert
import androidx.room.Query
import fastcampus.aop.part2.chapter04.model.History

@Dao
interface HistoryDao{

    @Query("SELECT * FROM history")
    fun getAll(): List<History>

    @Insert
    fun insertHistory(history: History)

    @Query("DELETE FROM history")
    fun deleteAll()

    @Delete
    fun  delete(history: History)

    @Query("SELECT * FROM history WHERE result LIKE :result")
    fun findByResult(result: String): List<History>
}
```

#### 4. 데이터베이스 추상 클래스 생성

db를 생성을 위해 추상 클래스를 생성해준다. 어노테이션에 연결할 dao와 버전을 속성으로 설정하고 함수를 선언한다.

```java
import androidx.room.Database
import androidx.room.RoomDatabase
import fastcampus.aop.part2.chapter04.dao.HistoryDao
import fastcampus.aop.part2.chapter04.model.History

//버전을 입력해야 마이그레이션을 위해서 작성
@Database(entities = [History::class], version = 1)
abstract class AppDatabase : RoomDatabase() {
    abstract fun historyDao(): HistoryDao
}
```

#### 5. 데이터베이스 사용

우선 빌더를 통해 생성을 해준다.

```java

lateinit var  db: AppDatabase

override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)

    db = Room.databaseBuilder(
        applicationContext,
        AppDatabase::class.java,
        "historyDb"
    ).build()
}
```

예시에서 사용된 것처럼 별도의 쓰레드를 구성하여 ui와 db의 저장이 같은 쓰레드에 돌지 않도록 처리해준다.

```java
fun historyClearButtonClicked(v: View){
    historyLinearLayout.removeAllViews()
    Thread(Runnable {
        db.historyDao().deleteAll()
    }).start()
}
```

### runOnUiThread

ui쓰레드와 db쓰레드가 다른 별도의 쓰레드이기 때문에 db쓰레드가 ui 메인 쓰레드의 View를 접근하기 위해서는 `runOnUiThread`를 사용해야 한다.

```java
fun historyButtonClicked(v: View) {
    historyLayout.isVisible = true
    historyLinearLayout.removeAllViews()

    Thread(Runnable {
        db.historyDao().getAll().reversed().forEach{
            runOnUiThread {
                val historyView = LayoutInflater.from(this).inflate(R.layout.history_row, null,false)
                historyView.findViewById<TextView>(R.id.expressionTextView).text = it.expression
                historyView.findViewById<TextView>(R.id.resultTextView).text = "= ${it.result}"

                historyLinearLayout.addView(historyView)
            }
        }
    }).start()
}
```

_위 historyView는 `LayoutInflater`를 사용하여 따로 생성한 xml 화면을 불러온다._
