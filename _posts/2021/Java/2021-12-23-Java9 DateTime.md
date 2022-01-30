---
layout: post
title:  "Java9 DateTime"
date:   2021-12-23
categories: [web]
---
# 날짜/시간

### java.time
java8 에서 도입한 java.time API는 이전의 API들의 결함을 해결하고 날짜/시간을 다룰 경우 권장하고 있다.

### 지역 날짜 (LocalDate) / 지역 시간 (LocalTime)
LocalDAte는 연월일이 포함된 날짜다. 정적메서드 `now`또는 `of`를 사용한다.

```java
LocalDate currentDate = LocalDate.now(); //오늘 날짜
LocalDate baseDate = LocalDate.of(2021,5,5);
LocalTime baseTime = LocalTime.now(); //현재 시간
LocalTime baseTime = LocalTime.of(22,30);
```

Chaining으로 메소드를 연결하여 객체를 다룰 수 있다.

```java
//일부 예시
LocalDate baseDate = LocalDate.of(2021,5,5);
System.out.println(baseDate.plusDays(99).plusMonths(2).minusDays(2)); //2021-10-10
```

### 날짜 조정기
종종 '매월 첫 번째 일요일' 같은 날짜를 계산할 때 사용할 수 있는 메서드가 있다.

`TemproalAdjuster`인터페이스를 상속받아 클래스 생성후 사용 할 수 있다.

우선 기본적으로 사용은 아래와 같이 할 수 있다.

```java
LocalDate baseDate = LocalDate.of(2021,5,5);
System.out.println(baseDate.with(TemporalAdjusters.lastDayOfMonth())); // 2021-05-31
```

```java
public class TheDayMartOff implements TemporalAdjuster {

    @Override
    public Temporal adjustInto(Temporal temporal) {
        //1. 계산을 위한 기준날짜
        LocalDate theDay = LocalDate.from(temporal);
        //2. 둘째 넷째 일요일을 구한다.
        LocalDate secondSunday = theDay.with(TemporalAdjusters.dayOfWeekInMonth(2, DayOfWeek.SUNDAY));
        LocalDate fourthSunday = theDay.with(TemporalAdjusters.dayOfWeekInMonth(4, DayOfWeek.SUNDAY));

        //3. 기준 날짜가 둘째 일요일 이전이면 둘째 일요일, 넷째 일요일 이전이면 넷째 일요일
        //   둘 다 아니면 다음달 둘째 일요일을 반환한다.
        if (theDay.isBefore(secondSunday)){
            return secondSunday;
        } else if (theDay.isBefore(fourthSunday)){
            return fourthSunday;
        } else {
            return theDay.plusMonths(1).with(TemporalAdjusters.dayOfWeekInMonth(2, DayOfWeek.SUNDAY));
        }
    }
}
```

위에서 선언한 클래스를 사용하면 아래와 같다.

```java
LocalDate baseDate = LocalDate.of(2021,5,5);
System.out.println(baseDate.with(new TheDayMartOff())); //위에서 선언한 TheDayMartOff() 클래스 생성
// 결과 : 2021-05-09
```

### Format
총 세가지 형식 지정 방식 있으나, 커스텀 지정방식만 살펴보면 아래와 같다.

```java
LocalDate baseDate = LocalDate.of(2021,5,5);
String formattedDate = DateTimeFormatter.ofPattern("G yyyy.MM.dd").format(baseDate);
System.out.println(formattedDate); // 서기 2021.05.05
```