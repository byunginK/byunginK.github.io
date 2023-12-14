---
layout: post
title: "Java Thread safe Collection"
date: 2023-10-27
categories: [spring]
---

# 자바 컬렉션 프레임워크, thread safe한 컬렉션

### 자바 컬렉션 프레임워크

List,Set,Map 인터페이스를 기준으로 여러 구현체가 존재한다. 이에 더해 Stack과 Queue 인터페이스도 존재한다.

컬렉션을 사용하는 이유는 여러 data를 다루는데 표준화된 클래스들을 제공해주기때문에 자료구조를 직접 구현하지 않아도 편하게 사용할 수 있기 때문이다. 또한, 배열과 다르게 컬렉션은 객체를 보관하기 위한 공간을 미리 정하지 않아도 되므로, 크기가 고정되어있지 않아 상황에 따라 객체의 수를 동적으로 정할 수 있다. 이는 프로그램의 공간적인 효율성 또한 높여준다.

### 자바 컬렉션 특징

    
    List : 순서가 있는 목록 List형, 중복값을 넣을 수 있고 순서가 존재한다.
    
    - ex) ArrayList, **Stack, Vector, CopyOnWriteArrayList**, LinkedList
---
    Set : 순서가 없는 집합 Set형, 중복값을 넣을 수 없다.
    
    - ex) HashSet, TreeSet, LinkedHashSet, **CopyOnWriteArraySet**
---
    Map : key,value으로 저장되는 Map형, key는 고유하지만 value는 중복해서 저장가능하다.
    
    - ex) HashMap, TreeMap, LinkedHashMap, **HashTable, ConcurrentHashMap**
---
    Stack과 Queue
    
    - ex ) **ConcurrentLinkedQueue, BlockingQueue**, Queue, Stack

### List

순서가 있는 목록 List형, 중복값을 넣을 수 있고 순서가 존재한다.

    
- ArrayList : 일반 배열과 비슷하나 크기가 동적  
- **Stack** : thread safe함. 내부적으로 모든 메서드에 synchronized가 붙어있기 때문이다. **Stack extends Vector**, 즉 Vector를 상속했는데 상속으로 인해 Vector를 통해 자식 클래스 Stack의 캡슐화가 깨진다.  
- **Vector** : thread safe함. 내부적으로 모든 메서드에 synchronized가 붙어있기 때문이다. ArrayList와 비슷하다.
- LinkeList : 연결 리스트 자료구조를 구현한 클래스. Queue,Deque,List의 구현체이기도 하다.  
    
- **CopyOnWriteArrayList** : thread safe하면서 병렬성을 확보해 성능이 좋다. 반복문에서 변경할 때마다 복사
    

### Set

순서가 없는 집합 Set형, 중복값을 넣을 수 없다.

- HashSet : 내부적으로 HashMap으로 구현되어있다. Map의 key - value 구조에서 key 대신에 value가 들어가고, value에는 더미 데이터가 들어갈 뿐이다. O(1)
- TreeSet : 값을 정렬하여 저장하는 set. red black tree로 값이 저장되는데 이는 이진트리로 균형잡힌 트리라서 삽입/삭제/탐색이 o(logN)이다.
- LinkedHashSet : 저장 순서를 보장하기 위해 사용되는 set
- **CopyOnWriteArraySet** : thread safe하면서 병렬성을 확보해 성능이 좋다.

### Map

key,value로 저장되는 Map형, key는 고유하지만 value는 중복해서 저장가능하다.

- HashMap : 키에 대한 hashCode 값에 따라 해시 버킷에 값을 저장한다?
    - key,value 둘 다 null이 허용되면 0번째 버킷을 찾아간다.
- **HashTable** :
    - thread safe함. 내부적으로 get, put, conains 메서드가 synchronized되어있기 때문이다.
- **ConcurrentHashMap** :
    - thread safe하면서 병렬성을 확보해 성능이 좋다. Lock Striping과 미약한 일관성 전략
- TreeMap : 키를 정렬하여 저장하는 Map
- LinkedHashMap : LinkedList로 구현된 HashMap. 키에 대한 저장 순서를 보장하기 위해서 사용된다.

### Queue

- **BlockingQueue** : thread safe함??
- **ConcurrentLinkedQueue** : thread safe함??

### HashMap은 null 키 값이 허용되는데 HashTable은 null 키,값이 허용되지 않는다.

```
public class HashMapKeyNullValue {

    Employee e1;

    public void display(){

        Employee e2=null;
        Map map=new HashMap();

        map.put(e2, "25");
        System.out.println("Getting the Value When e2 is set as KEY");
        System.out.println("e2 : "+map.get(e2));//e2 : 25
        System.out.println("e1 : "+map.get(e1));//e1 : 25
        System.out.println("null : "+map.get(null));//null : 25
// -> HashMap은 null 키에 대해 모두 같은 value 25를 반환.

        map.put(e1, "");
        System.out.println("Getting the Value when e1 is set as KEY");
        System.out.println("e2 : "+map.get(e2));//e2 :
        System.out.println("e1 : "+map.get(e1));//e1 :
        System.out.println("null : "+map.get(null));//null :
//-> HashMap은 null 키에 대해 모두 같은 value인 ""을 반환

        map.put(null, null);   // null as key and null as value
        System.out.println("Getting the Value when setting null as KEY and null as value");
        System.out.println("e2 : "+map.get(e2));//e2 : null
        System.out.println("e1 : "+map.get(e1));//e1 : null
        System.out.println("null : "+map.get(null));//null : null
//-> HashMap은 null 키에 대해 모두 같은 value인 null을 반환

        map.put(null, "30");
        System.out.println("Getting the Value when setting only null as KEY");
        System.out.println("e2 : "+map.get(e2));//e2 : 30
        System.out.println("e1 : "+map.get(e1));//e1 : 30
        System.out.println("null : "+map.get(null));//null : 30
//-> HashMap은 null 키에 대해 모두 같은 value인 30을 반환
    }
}
```

HashMap은 null을 key로 주면 0 해시버켓을 찾아간다. 그래서 null 키인 e1,e2,null 모두 0 해시버킷을 찾아가서 같은 값을 반환한다.

반면에 HashTable은 null 키나 값이 허용되지 않는다.

### Java HashMap은 어떻게 동작하는가?

### 동기화 컬렉션 클래스(Stack, Vector, HashTable)의 다중 연산 문제

Stack, Vector, HashTable 같은 동기화 컬렉션은 내부적으로 메서드에 synchronized가 걸려있다. 그래서 thread safe하다.

**하지만 동기화 컬렉션을 가지고 동기화 컬렉션의 여러 연산을 묶어 하나의 단일 연산으로 활용할 경우 문제가 발생한다.**

```
//개발자가 만든 메서드
public static Object getLast(Vector list) {
	int lastIndex = list.size() -1;
	return list.get(lastIndex);
}
//개발자가 만든 메서드
public static void delteteLast(Vector list) {
	int lastIndex = list.size() -1;
	list.remove(lastIdndex);
}

//반복문의 경우  ArrayIndexOutOfBoundesException이 터진다.
for (int i = 0; i< vector.size(); i++) {
	doSomething(vector.get(i));
}
```

- **반복 Iteration ( 컬렉션 내부의 모든 항목을 차례로 가져다 사용)** : 다중연산인 반복문으로 하나씩 데이터를 가져올 때 다른 스레드에서 데이터를 삭제하거나 변경해버리면 ArrayIndexOutOfBoundesException이 터진다. 게다가 containsAll, toString 등의 메서드 내부에서 반복문이 숨겨져있을 수 있다.
- **개발자가 만든 메서드** : 아래 코드에서 getLast, deleteLast 메서드를 두개의 스레드가 각각 동시 실행하면서 delete가 먼저 수행되서 삭제된 데이터를 조회하는 경우가 발생되면 ArrayIndexOutOfBoundesException이 터진다.
- **이동 navigation(특정한 순서에 맞춰 현재 보고 있는 항목의 다음 항목 위치로 이동함)**
- **없는 경우에만 추가하는 (컬렉션 내부에 추가하고자 하는 값이 있는지 확인하고, 기존에 갖고 있지 않은 경우에만 새로운 값을 추가하는 기능)**

```
public static Object getLast(Vector list) {
	synchronized(list) {
		int lastIndex = list.size() -1;
		return list.get(lastIndex);
	}
}

public static void delteteLast(Vector list) {
	synchronized(list) {
		int lastIndex = list.size() -1;
		list.remove(lastIdndex);
	}
}

synchronized(vector) {
	for (int i = 0; i< vector.size(); i++) {
		doSomething(vector.get(i));
	}
}
```

**해결 방법은 동기화 컬렉션 클래스 자체를 락으로 사용해 메서드를 동기화시키면 된다. 하지만 이는 성능적인 문제가 발생할 수 있다.**

특히 반복문에서 반복문을 실행하는 동안 vector 클래스 내부의 값을 변경하는 모든 스레드가 대기 상태에 들어간다. 즉, 반복문이 실행되는 동안에는 해당 vector를 사용하는 여러 스레드의 동시 작업을 다 막아버린다.

### Collection 클래스 : Collections.synchronizedXXX

```
Map m = Collections.synchronizedMap(new HashMap(...));
```

마치 HashTable 처럼 주요 메서드에 synchronized 키워드가 선언되어있다.

### Collection 클래스의 다중 연산 문제

Collection 클래스 역시 다중 연산 문제가 발생한다. Iterator와 for문 같은 반복문 문제가 있다. 게다가 containsAll, toString 등의 메서드 내부에서 반복문이 숨겨져있을 수 있다.

```
List<Widget> widgetList = Collections.synchronizedList(new ArrayList<>());
...
//ConcurrentModificationException이 발생
for (int i = 0; i< widgetList.size(); i++) {
	doSomething(widgetList.get(i));
}
```

반복문으로 하나씩 데이터를 가져올 때 다른 스레드에서 데이터를 삭제하거나 변경하는 작업을 막아주지는 못한다.

하지만 **fail fast 즉시 멈춤**의 형태로 반응한다. 즉시 멈춤이란 반복문을 실행하는 도중에 컬렉션 클래스 내부의 값이 변경되는 상황이 포착되면 그 즉시 ConcurrentModificationException이 발생하고 멈추는 처리 방법을 예기한다.

해결책은 역시나 반복문을 실행하는 코드 전체를 동기화하는 방법이 있다. 하지만 이도 역시 성능적으로 좋지 않다. 반복문이 실행되는 동안에는 해당 컬렉션을 사용하는 여러 스레드의 동시 작업을 다 막아버린다.

### 병렬 컬렉션 ( ConcurrentHashHashMap, CopyOnWriteArrayList, 컬렉션 클래스의 putifabsent 등등)

동기화 컬렉션 클래스, Collection 클래스 모두 내부적으로 메서드에 synchronized가 걸려있다. 따라서 이 클래스들을 사용하는 다중 연산 시에 문제가 발생할 수 있다. 문제를 해결 하기 위해 이 다중 연산 자체에 동기화를 할 수 있다. 하지만 이는 성능적으로 문제가 있다. 아예 해당 컬렉션들 사용하는 모든 스레드를 막아버리기 때문이다.

이 때 쓸 수 있는 것이 병렬 컬렉션 ConcurrentHashMap, ConpyOnWriteArrayList,ConpyOnWriteArraySet이 있다.

### ConcurrentHashMap

ConcurrentHashMap은 thread safe하면서 병렬성을 높여 hashtable, Collections.synchronizedMap보다 성능이 좋다.

이전까지는 모든 연산에서 단 하나의 락을 사용했기 때문에 특정 시점에 하나의 스레드만이 해당 컬렉션을 사용할 수 있다. 하지만 ConcurrentHashMap은 락 스트라이핑 **Lock Striping 덕분에 동시 읽기나 동시 쓰기에 높은 성능을 보인다.**

**Lock Striping**이란 컬렉션 데이터를 여러개의 범위로 나눠서 각 범위마다 특정 락이 담당하도록 하는 것이다. ConcurrentHashMap은 16개의 락을 배열로 마련해두고, 16개의 락 각자가 전체 해시 범위의 1/16에 대한 락을 담당한다. 덕분에 데이터의 각 영역이 서로 영향을 주지 않는 장업에 대해서는 경쟁이 일어나지 않기 때문에 여러 스레드에서 동시 접근이 가능하고 락 획득을 위한 대기 시간을 줄일 수 있다.

**또한 ConcurrentHashMap은 다중 연산인 반복문을 사용할 때에 즉시 멈춤 대신 미약한 일관성 전략을 취하기 때문에 ConcurrentModificationException이 발생하지 않는다.** 이전까지는 다중 연산인 반복문에서 즉시 멈춤으로 인해 ConcurrentModificationException이 터졌다.

**미약한 일관성 전략**이란 반복문을 사용할 때 컬렉션의 내용이 변경되어도, 반복분을 만들었던 시점의 상황대로 반복을 계속한다.

ConcurrentHashMap의 단점은

- size, isEmpty같은 메서드의 의미가 희미해진다는 것이다. 락도 여러개고 반복문 시에 데이터가 변경될 수도 있는 것이라서 연산 결과를 내는 시점에 이미 컬렉션 데이터가 변경될 수 있다는 것이다.
- 독점 사용을 못한다. SynchronziedMap, HashTable과 같이 독점적으로 락 하나로 사용하는 기능이 없다. 그래서 putifabsent와 같이 없을 경우에만 추가하는 연산같은 것을 통해 독점적으로 사용하는 것처럼 Atomicity가 보장되는 결과를 제공한다.

### CopyOnWriteArrayList

CopyOnWriteArrayList는 병렬성과 동시성을 확보한 클래스이다.

CopyOnWriteArrayList는 **변경할 때마다 복사하는 방식을 취해 ConcurrentModificationException이 발생하지 않는다.** 이전까지는 다중 연산인 반복문에서 즉시 멈춤으로 인해 ConcurrentModificationException이 터졌다.

변경할 때마다 복사하는 방식은

**컬렉션 내용이 벼경될 때마다 복사본을 새로 만들어낸다** 즉, 변경할 때마다 새로운 불변객체를 만드는 것이다. 반복문 Iterator를 뽑아내는 시점의 컬렉션 데이터를 기준으로 반복하며 만약 반복하는 동안에 컬렉션에 변경이 일어난다면 **반복문과는 상관없는 복사본을 대상으로 변경이 반영**되기 때문에 동시 사용에 문제가 없다.

### ConcurrentHashMap은 기존의 동기화된 HashMap과 비교해봤을 때 동일하게 동시성을 보장하면서도 성능은 높혔습니다. 어떻게 구현했길래 이게 가능했을까요?

### 동시성 측면에서 ConcurrentHashMap과 InnoDB 테이블과의 공통점을 설명해주실 수 있으실까요?

### Collections.synchronizedMap()의 내부 구현 방식과 ConcurrentHashMap의 내부 구현 방식에 대해 설명해주세요

### 배열과 ArrayList, LinkedList 의 차이점은 무엇인가요?

### 크기를 지정하지 않고 ArrayList 를 new 로 생성하면 크기 10의 ArrayList 가 생성됩니다. Array 는 크기를 넘길 수 없는데 반해 ArrayList 는 꽉 찬 List 에 element 를 추가로 더할 수 있습니다. 그렇다면 10개의 element 를 채워넣은 ArrayList 의 11번째 element 을 add 하기위해 어떤 일이 일어나는지 설명해주세요.

### 메소드에서 리스트 타입의 파라미터를 받을 때, ArrayList - List - Collection - Iterable 처럼 구체 타입 뿐 아니라 상위 타입도 받을 수 있습니다. 컬렉션을 받는 어떤 API 를 구현하실 때 구체 타입의 API 디자인을 선호하는지, 추상 타입의 API 디자인을 선호하는지를 설명해 주세요. 왜 그런 선택을 하시나요?

### 참고

자바 병렬 프로그래밍

[https://d2.naver.com/helloworld/831311](https://d2.naver.com/helloworld/831311)

[https://github.com/JaeYeopHan/Interview_Question_for_Beginner/tree/master/Java#collection](https://github.com/JaeYeopHan/Interview_Question_for_Beginner/tree/master/Java#collection)

[HashMap with Null Key and Null Value](https://stackoverflow.com/questions/25932730/hashmap-with-null-key-and-null-value)

[hashtable vs concurrenthashmap](https://roynus.tistory.com/672)

[Dayeon myeong](https://velog.io/@meme2367)
