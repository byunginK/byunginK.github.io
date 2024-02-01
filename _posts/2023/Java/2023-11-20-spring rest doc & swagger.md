---
layout: post
title: "Spring Rest Doc & Swagger 동시에 사용"
date: 2023-11-20
categories: [spring]
---
# 0. 시도하게된 이유

## 0.1 Swagger 경험

이전에 TODO List 라는 프로젝트에서 [Swagger 를 사용해본 적](https://github.com/jinan159/todo-list/commit/ac35b52912518af4beac5c0912b7ddef16f0e6d0)이 있습니다.

API 문서가 자동으로 생긴다는 점은 편했지만, 비즈니스 로직과 API 를 위한 코드가 섞여있다는 것이 마음에 들지 않았습니다.

## 0.2 Spring REST Docs 시도

그러던 중 [우아한형제들 기술블로그](https://techblog.woowahan.com/2597/)를 인상깊게 보아 Spring REST Docs 를 시도해보게 되었습니다.

Spring REST Docs 는 다음과 같은 장점이 있었습니다.

- API 문서가 테스트코드 통과 후 생성됨
- 비즈니스 로직에는 API 문서 관련 코드가 전혀 없음
- 커스텀이 자유로움

하지만, 다음과 같은 단점도 생겼습니다.

- 문서가 추가되면 asciidoc 을 문서를 일일이 편집해야함
- Swagger 에 있는 API 호출 기능을 사용할 수 없음
- (개인적) 별로 이쁘지 않은 UI

하지만 이런 단점보다 장점이 더 와닿아 Spring REST Docs 를 여러번 사용했습니다.

## 0.3 SwaggerUI + Spring REST Docs 가능성 확인

이 둘을 합칠 수 있는 방법이 있을지 찾아보던 중 [이 글](https://blog.jdriven.com/2021/10/generate-swagger-ui-from-spring-rest-docs/)을 발견했습니다.

요약하자면, 이 두 플러그인이 핵심입니다.

- `com.epages.restdocs-api-spec`
    - Spring REST Docs 의 결과물을 OpenAPI 3 스펙으로 변환(참고 : [OpenAPI 3](https://swagger.io/specification/))
- `org.hidetake.swagger.generator`
    - OpenAPI 3 스펙을 기반으로 SwaggerUI 생성(HTML, CSS, JS)

Spring REST Docs 는 글 작성일 기준 `asciidoc`, `markdown` 만 지원하는데요.

이 플러그인을 사용해서 Spring REST Docs 실행 결과를 OpenAPI 3 스펙을 출력하게 하고, 이를 이용해 SwaggerUI 를 생성하는 방식으로 동작합니다.

전체적인 흐름을 비교해보면 다음 그림과 같이 표현할 수 있습니다.

![SwaggerUI%20+%20Spring%20REST%20Docs%20%E1%84%92%E1%85%A1%E1%86%B7%E1%84%81%E1%85%A6%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5(feat%20%20c0f73f230ed2492ebae059f020774582/image.png](https://velog.velcdn.com/images/jinan159/post/7fc2e085-5a15-4ebe-a6b2-2ac589575ee2/image.png)

자세한 내용은 아래에서 프로젝트 생성과정부터 함께 살펴보겠습니다.

전체 소스코드가 궁금하시면 [여기](https://github.com/jinan159/example-codes/tree/main/restdocs-swagger)서 살펴보실 수 있습니다.

# 1. 프로젝트 생성

![SwaggerUI%20+%20Spring%20REST%20Docs%20%E1%84%92%E1%85%A1%E1%86%B7%E1%84%81%E1%85%A6%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5(feat%20%20c0f73f230ed2492ebae059f020774582/image%201.png](https://velog.velcdn.com/images/jinan159/post/18267b70-9ca2-4bd8-9a43-dba285458843/image.png)

Spring Web 과 Spring REST Docs 의존성을 추가하고, 프로젝트를 생성합니다.

## 1.1 (선택) Rest Assured 추가

저는 통합테스트를 작성할 예정이기 때문에, Rest Assured 를 추가하겠습니다.

```
...

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
-   testImplementation 'org.springframework.restdocs:spring-restdocs-mockmvc'
+   testImplementation 'org.springframework.restdocs:spring-restdocs-restassured'
+   testImplementation 'io.rest-assured:rest-assured'
}

...
```

# 2. Open API 3 스펙 출력 설정

Spring Rest Docs 의 결과로 **Open API 3 스펙**이 출력되도록 하기 위한 설정을 시작하겠습니다.

## 2.1 build.gradle 변경

```
// 2.1.1 restdocsApiSpecVersion 버전 변수 설정
+ buildscript {
+     ext {
+         restdocsApiSpecVersion = '0.16.2'
+     }
+ }

plugins {
    id 'org.springframework.boot' version '2.7.2'
    id 'io.spring.dependency-management' version '1.0.12.RELEASE'
// 2.1.2. asciidoctor 플러그인 제거, restdocs-api-spec 플러그인 추가
-   id 'org.asciidoctor.convert' version '1.5.8'
+   id 'com.epages.restdocs-api-spec' version "${restdocsApiSpecVersion}"
    id 'java'
}

group = 'com.example'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '17'

repositories {
    mavenCentral()
}

// 2.1.3
+ openapi3 {
+   setServer("http://localhost:8080")
+   title = "restdocs-swagger API Documentation"
+   description = "Spring REST Docs with SwaggerUI."
+   version = "0.0.1"
+   format = "yaml"
+ }

// 2.1.4 불필요한 설정 제거 (1)
- ext {
-     set('snippetsDir', file("build/generated-snippets"))
- }

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
// 2.1.5 restdocs-api-spec restassured 관련 의존성 추가
-   testImplementation 'org.springframework.restdocs:spring-restdocs-restassured'
+   testImplementation "com.epages:restdocs-api-spec-restassured:${restdocsApiSpecVersion}"
    testImplementation 'io.rest-assured:rest-assured'
}

tasks.named('test') {
// 2.1.4 불필요한 설정 제거 (2)
-    outputs.dir snippetsDir
    useJUnitPlatform()
}

// 2.1.4 불필요한 설정 제거 (3)
- tasks.named('asciidoctor') {
-     inputs.dir snippetsDir
-     dependsOn test
- }
```

### 2.1.1 restdocsApiSpecVersion 버전 변수 설정

`restdocs-api-spec` 라이브러리는 plugin 과 dependency 모두 사용할 예정입니다.

동일한 버전 설정을 편하게 하기 위해 버전 변수를 선언하여 설정합니다.

### 2.1.2 restdocs-api-spec 플러그인 추가

이 플러그인에는 Spring REST Docs 의 결과를 Open API 3 스펙으로 출력하는 **Gradle task** 가 포함되어 있습니다.

### 2.1.3 openapi3 설정

openapi3 로 Open API 3 스펙을 만들때 필요한 부가정보들 입니다.

- setServer(...) : 서버 주소를 설정합니다. API 요청 보내기 기능에서 이 주소가 사용됩니다.
- title : API 문서의 제목
- description : API 문서의 설명
- version : API 문서의 버전
- format : API 문서 출력 포멧(default : JSON)

### 2.1.4 불필요한 설정 제거

asciidortor 관련 불필요한 설정을 제거합니다.

### 2.1.5 restdocs-api-spec restassured 관련 의존성 추가

`restdocs-api-spec-restassured` 의존성을 추가합니다.

그리고 이 의존성에 이미 `spring-restdocs-restassured` 가 포함되어있기 때문에, 기존 의존성은 지워줍니다.

## 2.2 API 및 테스트 코드 작성

### 2.2.1 테스트용 API 작성

```
@RestController
public class SampleController {

    @PostMapping("/users")
    @ResponseStatus(HttpStatus.OK)
    public SampleResponse returnNameAndAge(@RequestBody SampleRequest request) {
        return new SampleResponse(request.name(), request.age());
    }
}

public record SampleRequest(String name, int age) {
}

public record SampleResponse(String name, int age) {
}
```

`POST /users` 로 요청 본문에 `name` 과 `age` 를 넘겨주면, 받은 정보를 그대로 반환하는 API 를 작성하였습니다.

### 2.2.2 테스트 코드 작성

위에서 작성한 API 에 대한 테스트 코드를 작성하겠습니다.

### 1) BaseControllerTest

```
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@ExtendWith(RestDocumentationExtension.class)
public abstract class BaseControllerTest {

    protected static final String DEFAULT_RESTDOC_PATH = "{class_name}/{method_name}/";

    protected RequestSpecification spec;

    @LocalServerPort
    int port;

    @BeforeEach
    void setUp() {
        RestAssured.port = port;
    }

    @BeforeEach
    void setUpRestDocs(RestDocumentationContextProvider restDocumentation) {
        this.spec = new RequestSpecBuilder()
                .setPort(port)
                .addFilter(documentationConfiguration(restDocumentation)
                        .operationPreprocessors()
                        .withRequestDefaults(prettyPrint())
                        .withResponseDefaults(prettyPrint()))
                .build();
    }
}
```

Rest Assured 와 API 스펙 관련 설정 코드의 중복을 제거하기 위해, `BaseControllerTest` 라는 추상 클래스를 선언합니다.

### 2) SampleControllerTest

```
class SampleControllerTest extends BaseControllerTest {

    private static final Snippet REQUEST_FIELDS = requestFields(
            fieldWithPath("name").type(JsonFieldType.STRING).description("이름"),
            fieldWithPath("age").type(JsonFieldType.NUMBER).description("나이")
    );

    private static final Snippet RESPONSE_FIELDS = responseFields(
            fieldWithPath("name").type(JsonFieldType.STRING).description("이름"),
            fieldWithPath("age").type(JsonFieldType.NUMBER).description("나이")
    );

    @Test
    void users_success_test() {
        String expectedName = "jay";
        int expectedAge = 27;

        JSONObject requestBody = new JSONObject();
        requestBody.put("name", expectedName);
        requestBody.put("age", expectedAge);

        given(this.spec)
            .filter(document(DEFAULT_RESTDOC_PATH, REQUEST_FIELDS, RESPONSE_FIELDS)) // API 문서 관련 필터 추가
            .accept(MediaType.APPLICATION_JSON_VALUE)
            .header("Content-type", "application/json")
            .body(requestBody)
            .log().all()

        .when()
            .post("/users")

        .then()
            .statusCode(HttpStatus.OK.value())
            .body("name", Matchers.equalTo(expectedName))
            .body("age", Matchers.equalTo(expectedAge));
    }
}
```

요청 본문으로 name, age 를 전달하고 응답에서 보낸 그대로 왔는지 확인합니다.

### 3) 문서 생성 확인

문서 생성 확인을 위한 Gradle Task 는 `openapi3` 입니다.

이 태스크는 2.1.2 에서 추가했던 `restdocs-api-spec` 플러그인이 apply 되며 생성됩니다..

(자세한 내용은 [RestdocsApiSpecPlugin.kt](https://github.com/ePages-de/restdocs-api-spec/blob/master/restdocs-api-spec-gradle-plugin/src/main/kotlin/com/epages/restdocs/apispec/gradle/RestdocsApiSpecPlugin.kt#L30)를 참고)

`openapi3` 는 Open API 3.0.1 문서를 yaml 포멧으로 `build/api-spec` 경로에 출력합니다.

그리고 `openapi3` 는 `check` 를 의존하도록 설정되어 있습니다.

![SwaggerUI%20+%20Spring%20REST%20Docs%20%E1%84%92%E1%85%A1%E1%86%B7%E1%84%81%E1%85%A6%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5(feat%20%20c0f73f230ed2492ebae059f020774582/img.png](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbP7AC0%2FbtrIEdoBJzY%2FjZhy1JYJkRVw1jr49829B1%2Fimg.png)

task 의존관계에 따라 `openapi3` 명령을 수행하면 compile -> test -> openapi3 가 실행되며, 문서가 생성될 것 입니다.

```
> ./gradlew openapi3
```

![SwaggerUI%20+%20Spring%20REST%20Docs%20%E1%84%92%E1%85%A1%E1%86%B7%E1%84%81%E1%85%A6%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5(feat%20%20c0f73f230ed2492ebae059f020774582/img%201.png](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FWLYy8%2FbtrIEXshPbd%2F0DiXlltaqkXWGOvrRsfrbk%2Fimg.png)

```
> Task :compileJava
> Task :processResources
> Task :classes
> Task :compileTestJava
> Task :processTestResources NO-SOURCE
> Task :testClasses
> Task :test
> Task :check
> Task :openapi3
```

위 과정을 거쳐 openapi3 작업까지 성공하였습니다.

![SwaggerUI%20+%20Spring%20REST%20Docs%20%E1%84%92%E1%85%A1%E1%86%B7%E1%84%81%E1%85%A6%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5(feat%20%20c0f73f230ed2492ebae059f020774582/img%202.png](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fby5o4r%2FbtrIxZloID7%2F0mLsO2KFi0JjFgGIhXK6AK%2Fimg.png)

위 사진 처럼 build/api-spec 경로에 openapi3.yaml 파일이 생성된 것을 보실 수 있습니다.

# 3. SwaggerUI 연동 추가

이전 단계에서 생성한 `openapi3.yaml` 으로 SwaggerUI 를 생성해보겠습니다.

추가되는 부분이 많으니 천천히 따라오시기 바랍니다.

```
+ import org.hidetake.gradle.swagger.generator.GenerateSwaggerUI
+ import org.springframework.boot.gradle.tasks.bundling.BootJar

buildscript {
    ext {
        restdocsApiSpecVersion = '0.16.2'
    }
}

plugins {
    id 'org.springframework.boot' version '2.7.2'
    id 'io.spring.dependency-management' version '1.0.12.RELEASE'
    id 'com.epages.restdocs-api-spec' version "${restdocsApiSpecVersion}"
// 3.1 swagger generator 플러그인 추가
+   id 'org.hidetake.swagger.generator' version '2.18.2'
    id 'java'
}

group = 'com.example'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '17'

repositories {
    mavenCentral()
}

// 3.2. swaggerSources 설정 추가
+ swaggerSources {
+   sample {
+       setInputFile(file("${project.buildDir}/api-spec/openapi3.yaml"))
+   }
+ }

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    testImplementation "com.epages:restdocs-api-spec-restassured:${restdocsApiSpecVersion}"
    testImplementation 'io.rest-assured:rest-assured'
// 3.3. Swagger 의존성 추가
+   swaggerUI 'org.webjars:swagger-ui:4.11.1'
}

tasks.named('test') {
    useJUnitPlatform()
}

// 3.4. Task 및 설정 추가
// 3.4.1
+ // GenerateSwaggerUI 태스크가, openapi3 task 를 의존하도록 설정
+ tasks.withType(GenerateSwaggerUI) {
+     dependsOn 'openapi3'
+ }
+
// 3.4.2
+ // 생성된 SwaggerUI 를 jar 에 포함시키기 위해 build/resources 경로로 로 복사
+ tasks.register('copySwaggerUI', Copy) {
+     dependsOn 'generateSwaggerUISample'
+
+     def generateSwaggerUISampleTask = tasks.named('generateSwaggerUISample', GenerateSwaggerUI).get()
+
+     from("${generateSwaggerUISampleTask.outputDir}")
+     into("${project.buildDir}/resources/main/static/docs")
+ }
+
// 3.4.3
+ // bootJar 실행 전, copySwaggerUI 를 실행하도록 설정
+ tasks.withType(BootJar) {
+     dependsOn 'copySwaggerUI'
+ }
```

## 3.1 swagger generator 플러그인 추가

생성된 API 스펙을 바탕으로 SwaggerUI 를 생성해주는 플러그인을 추가합니다.

## 3.2 swaggerSources 설정 추가

```
...

swaggerSources {
    sample {
        setInputFile(file("${project.buildDir}/api-spec/openapi3.yaml"))
    }
}

...
```

내 API 스펙이 있는 위치를 `이름`과 함께 지정합니다.

여기서는 `sample` 이 `이름` 이고, 그 안에 `sample` 의 `inputFile` 경로를 지정한 내용입니다.

이 경로는 `openapi3` task 로 생성된 API 스펙의 경로 입니다.

이렇게 설정하면, 사전에 지정된 prefix 와 `sample` 이라는 단어를 조합하여 CamelCase 로 여러가지 task 들이 생성됩니다.

우리가 사용할 예정인 `generateSwaggerUI~` task 는 [SwaggerGeneratorPlugin.groovy](https://github.com/int128/gradle-swagger-generator-plugin/blob/master/src/main/groovy/org/hidetake/gradle/swagger/generator/SwaggerGeneratorPlugin.groovy#L41) 에서 생성됩니다.

메소드를 따라가보면, prefix 뒤에 이름을 붙여 task 로 등록하는것을 알 수 있습니다.

정상적으로 설정이 되면, 다음 그림과 같이 생성된 task 를 확인하실 수 있습니다.

![SwaggerUI%20+%20Spring%20REST%20Docs%20%E1%84%92%E1%85%A1%E1%86%B7%E1%84%81%E1%85%A6%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5(feat%20%20c0f73f230ed2492ebae059f020774582/img%203.png](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fc1HXtG%2FbtrICOvsXQn%2FYkJfO7k7YSWKAvdtwq42O0%2Fimg.png)

- `generateSwaggerUISample` : swaggerSources 에서 설정한 특정 항목(sample)만 실행
- `generateSwaggerUI` : 등록된 모든 `generateSwaggerUI~` task 를 실행

## 3.3 Swagger 의존성 추가

이 설정에서 `swaggerUI` 라는 키워드는 [SwaggerGeneratorPlugin.groovy](https://github.com/int128/gradle-swagger-generator-plugin/blob/master/src/main/groovy/org/hidetake/gradle/swagger/generator/SwaggerGeneratorPlugin.groovy#L19) 에서 추가됩니다.

그리고 이렇게 추가된 dependency 는 [GenerateSwaggerUI.groovy](https://github.com/int128/gradle-swagger-generator-plugin/blob/master/src/main/groovy/org/hidetake/gradle/swagger/generator/GenerateSwaggerUI.groovy#L41) 에서 사용됩니다.

즉, API 스펙으로 SwaggerUI 를 만들어내는 `generateSwaggerUI` task 들에서 사용된다는 의미입니다.

## 3.4 Task 및 설정 추가

### 3.4.1

3.2 단계에서 설명한 task 들의 타입은 `GenerateSwaggerUI` 입니다.

이 task 들은 Open API 3 스펙 문서를 바탕으로 SwaggerUI 를 생성하기 때문에, `openapi3` task 에 의존하게 합니다.

그러면 API 스펙이 먼저 만들어지고, 그 다음 SwaggerUI 생성 작업이 시작되는 순서로 진행되게 됩니다.

### 3.4.2

`generateSwaggerUISample` 태스크는 SwaggerUI 를 `build/swagger-ui-sample` 경로에 생성합니다.

관련 설졍은 역시 [SwaggerGeneratorPlugin](https://github.com/int128/gradle-swagger-generator-plugin/blob/master/src/main/groovy/org/hidetake/gradle/swagger/generator/SwaggerGeneratorPlugin.groovy#L51) 에서 찾아볼 수 있습니다.

생성된 SwaggerUI 를 jar 에 같이 패키징하기 위해 `build/resources/main/static/docs` 경로로 복사합니다.

### 3.4.3

`bootJar` 는 **실행 가능한 jar** 파일을 만드는 task 입니다.

이때 `build/resources` 경로의 리소스 파일들이 jar 파일에 함께 패키징 됩니다.

## 3.5 UI 생성 확인

```
./gradlew build

----

> Task :compileJava
> Task :processResources
> Task :classes
> Task :bootJarMainClassName
> Task :jar
> Task :compileTestJava
> Task :processTestResources NO-SOURCE
> Task :testClasses
> Task :test
> Task :check
> Task :openapi3
> Task :generateSwaggerUISample
> Task :copySwaggerUI
> Task :bootJar
> Task :assemble
> Task :build
```

task 가 위와 같은 순서로 잘 실행되는것을 확인할 수 있습니다.

그리고 `build/swagger-ui-sample` 에 생성된 SwaggerUI 가 `build/resources/main/static/docs` 로 잘 복사된것도 아래 그림처럼 확인할 수 있습니다.

![SwaggerUI%20+%20Spring%20REST%20Docs%20%E1%84%92%E1%85%A1%E1%86%B7%E1%84%81%E1%85%A6%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5(feat%20%20c0f73f230ed2492ebae059f020774582/img%204.png](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbQd1aq%2FbtrIC0b72FO%2FWB6B9UhizJnYP1yUMxDzuk%2Fimg.png)

## 3.6 SwaggerUI 확인

bootRun 명령어를 통해, 이전 단계에서 빌드된 jar 을 파일을 실행 합니다.

```
./gradlew bootRun
```

그리고 [http://localhost:8080/docs/index.html](http://localhost:8080/docs/index.html)로 접속하면 SwaggerUI 를 확인하실 수 있습니다.

![SwaggerUI%20+%20Spring%20REST%20Docs%20%E1%84%92%E1%85%A1%E1%86%B7%E1%84%81%E1%85%A6%20%E1%84%89%E1%85%A1%E1%84%8B%E1%85%AD%E1%86%BC%E1%84%92%E1%85%A1%E1%84%80%E1%85%B5(feat%20%20c0f73f230ed2492ebae059f020774582/img%205.png](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbWuuNp%2FbtrIBTEjMvU%2Fme8YfjbLYkOok40YR0DyK0%2Fimg.png)