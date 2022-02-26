---
layout: post
title: "android basic 10"
date: 2022-02-26
categories: [Kotlin]
---

# UI

### ViewPager2

스와이프 뷰를 사용하면 손가락의 가로 동작이나 스와이프로 탭과 같은 동위 화면 간을 탐색할 수 있다.

```xml
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <androidx.viewpager2.widget.ViewPager2
        android:id="@+id/viewPager"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

    <ProgressBar
        android:id="@+id/progressBar"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center" />
</FrameLayout>
```

`ViewPager2`를 추가하고 사용할 수 있다.

##### custom transform

기본 화면 슬라이드 애니메이션과 다른 애니메이션을 표시하려면 ViewPager2.PageTransformer 인터페이스를 구현하여 ViewPager2 객체에 제공

화면에 표시되지 않는 인접 페이지에 대해 한 번씩 호출됩니다. 예를 들어, 페이지 3이 표시되고 사용자가 페이지 4로 드래그하면 동작의 각 단계에서 페이지 2, 3, 4에 대해 transformPage()가 호출한다.

페이지를 결정하여 맞춤 슬라이드 애니메이션을 만들 수 있다.

페이지가 화면을 채우면 위치 값은 0입니다. 페이지가 화면 오른쪽에서 벗어나면 위치 값은 1입니다. 사용자가 페이지 1과 페이지 2의 중간으로 스크롤하면 페이지 1의 위치 값은 -0.5이고 페이지 2의 위치 값은 0.5입니다. 화면의 페이지 위치에 따라 setAlpha(), setTranslationX() 또는 setScaleY()와 같은 메서드로 페이지 속성을 설정하여 맞춤 슬라이드 애니메이션을 만들 수 있다.

```java
private fun initViews() {
    //페이지가 넘어갈때 custom하여 사용할 수 있다.
    viewPager.setPageTransformer { page, position ->
        when{
            position.absoluteValue >= 1F -> {
                page.alpha = 0F
            }
            position == 0F -> {
                page.alpha = 1F
            }
            else -> {
                page.alpha = 1F - 2 * position.absoluteValue
            }
        }
    }
}
```

##### 무한 스크롤링

첫번째 페이지를 중간으로 지정하고 스와프하여 position의 값을 전체 사이즈로 나눈 나머지값으로 페이지를 보이도록 설정하여 계속 돌도록 설정 한다.

**첫 랜딩 페이지를 중간으로 설정**

```java
private fun displayQuotesPager(quotes: List<Quote>, isNameRevealed: Boolean) {

    val adapter = QuotesPagerAdapter(
        quotes = quotes,
        isNameRevealed = isNameRevealed
    )
    //첫 페이지를 가운데로 지정하고 첫화면에서 좌측으로도 무한 스크롤되도록 설정
    viewPager.adapter = adapter
    viewPager.setCurrentItem(adapter.itemCount/2, false)
}
```

**recyclerVeiw adpter부분**

```java
override fun onBindViewHolder(holder: QuoteViewHolder, position: Int) {
    //무한 스크롤
    val actualPosition = position % quotes.size
    holder.bind(quotes[actualPosition], isNameRevealed)
}

override fun getItemCount() = Int.MAX_VALUE
```

# Back

### RecyclerView

RecyclerView를 사용하면 대량의 데이터 세트를 효율적으로 표시할 수 있다. <br>
RecyclerView는 이러한 개별 요소를 재활용합니다. 항목이 스크롤되어 화면에서 벗어나더라도 RecyclerView는 뷰를 제거하지 않습니다. 대신 RecyclerView는 화면에서 스크롤된 새 항목의 뷰를 재사용합니다. 이렇게 뷰를 재사용하면 앱의 응답성을 개선하고 전력 소모를 줄이기 때문에 성능이 개선된다.

`RecyclerView`는 `뷰 홀더`객체로 이루어진 그룹이다.뷰 홀더가 생성되었을 때는 뷰 홀더에 연결된 데이터가 없습니다. 뷰 홀더가 생성된 후 RecyclerView가 뷰 홀더를 뷰의 데이터에 바인딩 한다. <br>`RecyclerView.ViewHolder`를 사용하여 정의 가능

RecyclerView는 뷰를 요청한 다음, 어댑터에서 메서드를 호출하여 뷰를 뷰의 데이터에 바인딩한다. <br>
`RecyclerView.Adapter`를 확장하여 어댑터를 정의

##### RecyclerView.Adapter

1. onCreateViewHolder(): RecyclerView는 ViewHolder를 새로 만들어야 할 때마다 이 메서드를 호출합니다. 이 메서드는 ViewHolder와 그에 연결된 View를 생성하고 초기화하지만 뷰의 콘텐츠를 채우지는 않습니다. ViewHolder가 아직 특정 데이터에 바인딩된 상태가 아니기 때문이다.
2. onBindViewHolder(): RecyclerView는 ViewHolder를 데이터와 연결할 때 이 메서드를 호출합니다. 이 메서드는 적절한 데이터를 가져와서 그 데이터를 사용하여 뷰 홀더의 레이아웃을 채웁니다.
3. getItemCount(): RecyclerView는 데이터 세트 크기를 가져올 때 이 메서드를 호출합니다. RecyclerView는 이 메서드를 사용하여, 항목을 추가로 표시할 수 없는 상황을 확인합니다.

**아래는 RecyclerView.Adapter를 선언 예시**

```java
class QuotesPagerAdapter(
    private val quotes: List<Quote>,
    private val isNameRevealed: Boolean
): RecyclerView.Adapter<QuotesPagerAdapter.QuoteViewHolder>() {


    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int) =
        QuoteViewHolder(
            LayoutInflater.from(parent.context)
                .inflate(R.layout.item_quote, parent, false)
        )

    override fun onBindViewHolder(holder: QuoteViewHolder, position: Int) {
        //무한 스크롤
        val actualPosition = position % quotes.size
        holder.bind(quotes[actualPosition], isNameRevealed)
    }

    override fun getItemCount() = Int.MAX_VALUE

    class QuoteViewHolder(itemView: View):RecyclerView.ViewHolder(itemView){

        private val quoteTextView: TextView = itemView.findViewById(R.id.quoteTextView)
        private val nameTextView: TextView = itemView.findViewById(R.id.nameTextView)

        @SuppressLint("SetTextI18n")
        fun bind(quote: Quote, isNameRevealed: Boolean){
            quoteTextView.text = "\"${quote.quote}\""

            if(isNameRevealed) {
                nameTextView.text = "- ${quote.name}"
                nameTextView.visibility = View.VISIBLE
            }else{
                nameTextView.visibility = View.GONE
            }
        }
    }
}
```

[전체 코드 보러 가기](https://github.com/byunginK/Andriod_Project/tree/main/chapter10)
