---
layout: post
title:  "Java9 Exception"
date:   2021-12-16
categories: [web]
---
# Exception
### try-with-resources

예외처리가 필요한 이유 중 하나는 리소스관리 때문이다. 때문에 try구문에 리소스를 지정하여 try 블록의 끝에 어느 경우에든 리소스 객체의 close 메서드가 호출 되게 한다.
(리소스는 반드시 AutoCloseable 인터페이스를 구현하는 클래스에 속해야한다.)

```java
Path path = Paths.get("yesterda.txt");

//try with resource 구문
try (Scanner in = new Scanner(path, "UTF-8");){

    in.useDelimiter("\n");
    while (in.hasNext()){
        System.out.println(in.next());
    }
}
```

위와 같이 작성하게 되면 finally 없이 리소스 해제가 가능하다.

```java
/* try with resource구문을 사용하면 내부적으로 close 해주기 때문에 finally를 사용할 필요가 없다.
finally {
    if(in != null) in.close();
}
*/
```

### 사용자 정의 예외처리
직접 예외 처리를 작성하여 처리 할 수 있다.

아래 코드의 `LylicReader`클래스의 `catch`는 `IOException`의 예외가 발생 했을때 `BizException`을 던지도록 되어있다.

```java
public class LylicReader {
    public void doJob(){
        Path path = Paths.get("yesterda.txt");

        //try with resource 구문
        try (Scanner in = new Scanner(path, "UTF-8");){

            in.useDelimiter("\n");
            while (in.hasNext()){
                System.out.println(in.next());
            }
        } catch (IOException e) {
            //사용자 정의 예외처리 사용
            throw new BizException("파일이 없습니다.", e);
        }
    }
}
```

`BizException`은 아래와 같이 구성하였다.

```java
public class BizException extends RuntimeException {
    public BizException() {
        super();
    }

    public BizException(String message) {
        super(message);
    }

    public BizException(String message, Exception cause) {
        super(message, cause);
    }

    public BizException(Exception cause) {
        super(cause);
    }
}
```

그리고 최종적으로 사용은 `rethrows`를 사용하여 적용하였다.

```java
public class ExceptionTest {
    public static void main(String[] args) {
        LylicReader reader = new LylicReader();
        try {
            reader.doJob();
        //최종적으로 사용자 예외처리를 적용하여 LylicReader의 rethrows를 사용한다.
        }catch (BizException e){
            System.out.println(e); //com.acompany.exceptions.BizException: 파일이 없습니다.
        }
    }
}
```