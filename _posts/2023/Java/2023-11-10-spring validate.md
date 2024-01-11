---
layout: post
title: "Spring Validate"
date: 2023-11-10
categories: [spring]
---
# 검증 - Validator

## Validator란,

BindingResult, FieldError, ObjectError를 통해 Controller에서 값을 검증할 수 있지만, 이 경우 Controller의 크기가 너무 커지고 단일 책임 원칙에 위배된다.

그래서 Spring에서는 Validator라는 인터페이스를 통해 별도의 검증용 Spring Bean을 만들 수 있도록 기능을 제공한다.

프로그래머는 Validator 인터페이스를 상속받아 구현체를 만들어 검증 로직을 구현할 수 있다. 그리고 Spring Bean으로 만들어지기 때문에, Controller에서는 해당 구현체를 주입받아 사용할 수 있게 된다.

## Gradle

```
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-validation'
}
```

Validator를 사용하기 위해선 위처럼 `build.gradle`에 등록해야한다.

## Validator Interface

```java
package org.springframework.validation;

public interface Validator {
    boolean supports(Class<?> clazz);
    void validate(Object target, Errors errors);
}
```

- `boolean supports()`
    - 해당 Validator의 구현체가 해당 Object Type의 에러를 지원하는지 알려주기 위한 함수이다.
    - `return Domain.class.isAssignableFrom(clazz);`
- `void validate()`
    - 검증 로직을 구현하는 함수이다.
    - `Object target`: 검증 대상 객체
    - `Errors errors`: BindingResult

## Errors Interface

```java
package org.springframework.validation;

public interface Errors {
    String NESTED_PATH_SEPARATOR = PropertyAccessor.NESTED_PROPERTY_SEPARATOR;

    void reject(String errorCode);
    void reject(String errorCode, String defaultMessage);
    void reject(String errorCode, @Nullable Object[] errorArgs, @Nullable String defaultMessage);

    void rejectValue(@Nullable String field, String errorCode);
    void rejectValue(@Nullable String field, String errorCode, String defaultMessage);
    void rejectValue(@Nullable String field, String errorCode,
            @Nullable Object[] errorArgs, @Nullable String defaultMessage);
}
```

### reject

- `String errorCode`
    - `errors.properties`에 저장된 Key 값을 찾는 용도로 사용된다.
- `@Nullable Object[] errorArgs`
    - errorCode를 이용해 찾은 Value 값에 인자를 치환하기 위한 용도로 사용된다.
- `@Nullable String defaultMessage`
    - errorCode를 이용해 Key 값을 찾을 수 없을 때 사용되는 기본 메시지이다.
- ObjectError를 생성하여 BindingError에 넣는다.

### rejectValue

- errorCode, errorArgs, defaultMessage는 reject와 같다.
- `@Nullable String field`
    - 에러가 발생한 field의 이름을 전달할 때 사용된다.
- FieldError를 생성하여 BindingError에 넣는다.

## Controller 주입 - 예제

Validator 인터페이스를 상속받아 구현체를 작성하였다면, Controller에 주입시켜 사용할 수 있는데, 이는 예제를 통해 알아보자.

### Item

```java
@Getter
@RequiredArgsConstructor
public class Item {
    private final String itemName;
    private final Integer price;
    private final Integer quantity;
}
```

단순히 값 검사와 클라이언트의 요청을 받기 위한 데이터 객체이다.

- `@Getter`
    - 해당 객체의 모든 field에 대한 Getter 메서드 생성
- `@RequiredArgsConstructor`
    - 해당 객체의 `final` 키워드가 붙은 모든 field에 대한 생성자 생성

### ItemValidator

```java
@Component
public class ItemValidator implements Validator {
    /**
     * Item 타입과 clazz 타입이 같은지 검사
     */
    @Override
    public boolean supports(Class<?> clazz) {
        return Item.class.isAssignableFrom(clazz);
    }

    /**
     * 검증 로직
     *
     * @param target 유효성을 검사할 객체
     * @param errors BindingResult 의 부모 객체
     */
    @Override
    public void validate(
            Object target, Errors errors
    ) {
        Item item = (Item) target;

        // 검증 로직
        // itemName != null && itemName != " "
        ValidationUtils.rejectIfEmptyOrWhitespace(errors, "itemName", "required");

        if (item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1000000) {
            errors.rejectValue("price", "range", new Object[]{1000, 1000000}, null);
        }

        if (item.getQuantity() == null || item.getQuantity() >= 9999) {
            errors.rejectValue("quantity", "max", new Object[]{9999}, null);
        }

        // 특정 필드가 아닌 복합 룰 검증
        if (item.getPrice() != null && item.getQuantity() != null) {
            int resultPrice = item.getPrice() * item.getQuantity();
            if (resultPrice < 10000) {
                errors.reject("totalPriceMin", new Object[]{10000, resultPrice}, null);
            }
        }
    }
}
```

- `boolean supports()`
    - 검사하고 싶은 대상이 Item 객체인지 확인을 위한 메서드이다.
    - `return Item.class.isAssignableFrom(clazz);`
- `public void validate(Object *target*, Errors *errors*)`
    - supports 메서드를 통과했다면, 실행되는 검증 로직 메서드이다.
    - 현재 적용된 검증 로직은 다음과 같다.
        - itemName field가 빈 값인지 검증 (`””`, `“ “`, `null`)
        - price field가 빈 값인지, 1,000 이하 1,000,000 이상 인지 검증
        - quantity field가 빈 값인지, 9,999 이상 인지 검증
        - price + quantity 의 값이 10,000 이하 인지 검증

### 주입 및 등록

```java
@RestController
public class ValidationController {
    private final ItemValidator itemValidator;

    @Autowired
    public ValidationController(
            ItemValidator itemValidator
    ) {
        this.itemValidator = itemValidator;
    }

    @InitBinder
    public void init(
            WebDataBinder dataBinder
    ) {
        dataBinder.addValidators(itemValidator);
    }
}
```

- `WebDataBinder.addValidators`
    - addValidators 메서드를 통해 Validator 구현체를 등록할 수 있다.
- `@InitBinder`
    - [공식 Docs](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/bind/annotation/InitBinder.html)
    - 해당 Controller에만 적용되는 설정을 수행할 수 있다.
    - 해당 Controller의 모든 요청에 대하여 요청 전에 실행된다.
        - 만약, 각 요청에 대해 따로 실행되도록 설정하려면 value 파라미터에 key 값을 입력하면 된다.
        - `@InitBinder("key")`
    - `@RequestMapping` 에서 지원하는 Arguments를 모두 지원한다.
    - 해당 애노테이션을 적용받는 메서드는 반환값이 존재하면 안된다. (void)
    - 일반적인 인수는 [WebRequest](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/context/request/WebRequest.html), [WebDataBinder](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/bind/WebDataBinder.html) 이다.

### 사용

```java
@RestController
public class ValidationController {
    // ...

    @PostMapping("/valid")
    public String valid(
            @Validated @RequestBody Item item,
            BindingResult bindingResult
    ) throws Exception {
        if (bindingResult.hasErrors()) {
            throw new ValidationException("validation failed");
        }

        return "validation ok";
    }
}
```

- `@Validated`
    - Spring에서 제공하는 애노테이션이다.
        - `org.springframework.validation.annotation`
    - Spring에서 `@Validated`를 지원하기 전에는 Java에서 제공하는 `@Valid`를 사용했다.
    - 검증이 필요한 객체가 있다면, 해당 메서드의 Argument 앞에 `@Validated` 애노테이션을 붙이면 된다.

### 프로그램의 흐름

클라이언트의 요청이 오면 다음의 순서를 따른다.

- DispatcherServlet 실행
- ItemValidator.supports 메서드 실행
- ItemValidator.validate 메서드 실행
- 검증 결과를 Errors에 쌓는다.
- 검증이 완료되면 Errors를 BindingResult 형태로 해당 메서드에 전달
- ValidationController.valid 메서드 실행
- 검증 단계에서 Error 가 발생했다면, `ValidationException("validation failed");` 발생
- 무사히 통과되었다면, `"validation ok"` 를 클라이언트에게 전달.

## 정리

검증을 위한 인터페이스와 등록방법을 알아보았다.

하지만, 이 방법은 클라이언트의 요청에 대한 정밀한 검증이 필요할 때에는 유용할지 몰라도, 일반적으로 사용되고 자주 사용되는 검증은 정해져 있기 마련이다.

그런 간단한 검증마저도 모든 요청에 대해 개별적인 Validator를 등록하기에는 무리가 있다.

그래서 Bean Validation에서는 각종 일반적인 검증에 대해 애노테이션 형태로 사용할 수 있도록 제공한다.