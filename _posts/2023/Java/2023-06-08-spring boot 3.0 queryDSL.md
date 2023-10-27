---
layout: post
title: "Spring boot 3.0 QueryDSL"
date: 2023-06-08
categories: [JAVA]
---

# Spring boot 3.0 이상 gradle QueryDSL설정

### build.gradle

```groovy
plugins {
	id 'java'
	id 'org.springframework.boot' version '3.0.3'
	id 'io.spring.dependency-management' version '1.1.0'
	//querydsl 추가
	id "com.ewerk.gradle.plugins.querydsl" version "1.0.10"
}

group = 'com'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '17'

configurations {
	compileOnly {
		extendsFrom annotationProcessor
	}
}

repositories {
	mavenCentral()
}

dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
	implementation 'org.springframework.boot:spring-boot-starter-security'
	implementation 'org.springframework.boot:spring-boot-starter-web'
	implementation 'org.springframework.boot:spring-boot-starter-validation'

	// QueryDSL 설정
	implementation "com.querydsl:querydsl-jpa:5.0.0:jakarta"
	annotationProcessor "com.querydsl:querydsl-apt:5.0.0:jakarta"
	annotationProcessor "jakarta.annotation:jakarta.annotation-api"
	annotationProcessor "jakarta.persistence:jakarta.persistence-api"
	// -- QueryDSL ---

	compileOnly 'org.projectlombok:lombok'
	runtimeOnly 'com.h2database:h2'
	annotationProcessor 'org.projectlombok:lombok'
    
	testImplementation 'org.projectlombok:lombok:1.18.22'
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
	testImplementation 'org.springframework.restdocs:spring-restdocs-mockmvc'
	testImplementation 'org.springframework.security:spring-security-test'

	asciidoctorExt 'org.springframework.restdocs:spring-restdocs-asciidoctor'
	implementation 'com.auth0:java-jwt:4.3.0'

}

tasks.named('test') {
	useJUnitPlatform()
}

// Querydsl 설정부
def generated = 'src/main/generated'

querydsl {
	jpa = true
	querydslSourcesDir = generated
}
sourceSets {
	main.java.srcDir generated
}

compileQuerydsl{
	options.annotationProcessorPath = configurations.querydsl
}

configurations {
	compileOnly {
		extendsFrom annotationProcessor
	}
	querydsl.extendsFrom compileClasspath
}
```

### ****compileQuerydsl에 clean 동작 추가****

radle의 **complieQuerydsl** task를 통해 QType을 생성하는데, 기존 QType이 존재하면 에러가 발생했다.

따라서, **clean** 이후 c**omplieQuerydsl** 을 실행시켜주어야 했는데 매번 두 번 클릭하기 귀찮았기 때문에 **clean** 동작을 추가해주었다.

```groovy
// complieQuerydsl Task에 clean 동작 추가
tasks.compileQuerydsl.dependsOn(clean);

compileQuerydsl{
	options.annotationProcessorPath = configurations.querydsl
}
```

### ****Build 시 QType으로 인해 문제가 생길 경우****

QueryDsl을 도입하고 나서 몇몇 gradle task를 실행할 때 간헐적으로 에러가 발생하는 상황이 빈번하게 발생했다.

```groovy
plugins {
    // querydsl plugin 제거
	// id "com.ewerk.gradle.plugins.querydsl" version "1.0.10"
}

// Querydsl 설정부
def generatedDir = 'src/main/generated'

clean {
	delete file (generatedDir)
}

//querydsl {
//	jpa = true
//	querydslSourcesDir = generated
//}
//sourceSets {
//	main.java.srcDir generated
//}
//compileQuerydsl{
//	options.annotationProcessorPath = configurations.querydsl
//}
//configurations {
//	compileOnly {
//		extendsFrom annotationProcessor
//	}
//	querydsl.extendsFrom compileClasspath
//}
```

위 QueryDsl 관련 설정 코드에서 변화가 있는 부분만 추출했다.

dependency는 유지하고 Querydsl 관련 플러그인 제거 및 clean 동작에 QType 제거 동작을 추가해주면 gradle task - build시 새 QType이 생성된다.