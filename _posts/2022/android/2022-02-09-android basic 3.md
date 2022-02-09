---
layout: post
title: "android basic 3"
date: 2022-02-09
categories: [Kotlin]
---

## constraintVertical_bias / constraintHorizontal_bias

보통 `app:layout_constraintEnd_toEndOf="parent"`와`app:layout_constraintStart_toStartOf="parent"`를 사용하여 레이아웃의 중앙에 위치 하도록 한다.

단, bias를 사용하면 수직/수평의 위치에서 한쪽으로 더 비중을 크게 만들 수 있다. 예를 들어 기본 50%대신 30% bias의 값을 설정하게 되면 왼쪽이 더 짧아지게 만들 수 있다. (그림 5 참고)

```xml
<androidx.constraintlayout.widget.ConstraintLayout ...>
    <Button android:id="@+id/button" ...
        app:layout_constraintHorizontal_bias="0.3"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"/>
```

![image](https://user-images.githubusercontent.com/65350890/153204899-ae6e00d6-4011-4d29-b494-bebdf37b65d7.png)

## AppCompatButton

`button`의 하위 클래스로 android 버전에서 앱에 최신 테마를 적용하는데 도움을 준다. 따라서 button의 `background` 및 테마를 적용하게 해준다.
