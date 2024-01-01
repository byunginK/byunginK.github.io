---
layout: post
title: "Spring log"
date: 2023-11-09
categories: [spring]
---

# Springboot Log 남기기


## **1. Log란?**

**사전적 의미로는 It에서 발생되는 모든 행위와 이벤트 정보를 시간에 따라 남겨둔 데이터를 지칭한다.**

**간단하게 이야기하면, 로그는 일정의 표시이고, 기록입니다. 로그를 통해서 애플리케이션의 상태를 관찰할 수도 있고, 오류가 발생한 부분에 대해서 인지할 수 있습니다. 즉 로직의 흐름, 예외 등등을 파악할 수 있도록 인지하기 할 수 있고, 로그를 통해서 서비스의 품질을 관리할 수도 있습니다.**

**예를 들어서 서비스를 이용하는 사용자가 자주 사용하는 기능이라던지 혹은 사람들이 어떠한 게시물을 열람할 때 로그 남기도록 만들었습니다. 그렇다면 서비스를 제공하는 입장에선 로그라는 데이터를 이용해서 기능을 더 발전시키거나, 사용자의 니즈를 찾아 품질이 높은 서비스를 제공할 수 있습니다. 
하지만 로그를 남기는 것 또한 비용이 발생하기 때문에 적절한 상황, 적절한 로그를 작성하는 것이 중요합니다. 적절한 상황이라는 것은 애플리케이션이 어떠한 행동을 하는지에 초점을 맞춘다면, 쉽게 찾을 수 있습니다. 로그 작성하는 법은 사람에 따라 다릅니다. 하지만 안 좋은 예와 좋은 예는 명확합니다. 하나씩 알아보겠습니다.**

### **1-1. 좋지 않은 로그**

```
try{	// 중략}catch (Exception e){    log.error("{}",e.getClass());	// 중략}
```

**로직에서 예외가 발생했을 때 로그를 기록하는 것은 좋습니다. 하지만 로그라는 것은 애플리케이션의 상태를 관찰하는 것이라고 위에서 살펴봤습니다. 위와 같이 단순히 예외가 발생했을 때 예외만 출력한다면 어떠한 예외가 왜 발생했는지 알 수 없습니다. 물론 코드의 양이 적고, 자신이 작성한 코드라면 알 수도 있습니다. 대부분의 상황이 그렇지 않기 때문에 로그는 누가 봐도 의미를 파악할 수 있도록 해주는 것이 중요합니다.**

### **1-2. 위보다는 좋은 로그**

```
try{}catch (Exception e){    log.error("userId={},uri ={}, method = {}, parameter={}, time={}, errorMessage={}",            userId,uri,method,parameter,time,e.getMessage());}
```

**1-1에서 살펴본 로그와는 다르게 많은 정보를 담고 있습니다. 누가, 어떤 경로로, 호출한 메서드와 파라미터, 언제, 발생한 이유 같이 상세하게 로그를 남긴다면, 로그의 의미를 쉽게 파악할 수 있습니다.**

## **2. Springboot 로그 사용**

**로그란 무엇인지 로그를 왜 사용하는지에 대해서 알아봤습니다. 이제는 Springboot에서 로그를 남기는 법에 대해서 알아보겠습니다.**

**이전에는 log4j라는 것을 사용해서 로그를 남겼습니다. 하지만 개발이 중단됐고, 성능이 더 좋은 logback이 등장해 저희는 이것을 사용합니다.**

### **2-1. logback이란?**

**자바의 오픈소스 로깅 프레임워크입니다. 앞으로 저희가 사용할 Slf4j의 구현체입니다.**

**Springboot에서는 spring-boot-starter-logging안에 기본적으로 포함되어 있어서 따로 dependency를 추가하지 않고 사용 가능합니다.**

### **2-2. 로그 사용**

**로그를 선언하고, 사용하는 방법에는 2가지가 있습니다.**

**1. 로그를 사용할 클래스 위에 @Slf4j 붙이기(Lombok을 사용해야 합니다.)**

```
@Slf4jpublic class Log{}
```

**2. LoggerFactory에서 직접 가져오기**

```
private static final Logger log = LoggerFactory.getLogger(Log.class);
```

### **2-3. Springboot 로그 설정**

**1. resourece 디렉터리 밑에 logback-spring.xml을 읽고, 로그를 만듭니다.**

**2. 위의 logback-spring.xml이 없다면,. yml파일 혹은. properties 파일을 읽고, 로그를 만듭니다.**

**3. 만약 두 개 다 있다면,. yml파일 먼저 읽고, logback-spring.xml 파일에서 추가되는 부분을 사용합니다.**

**저희는. yml 파일로 작성할 예정이며, profile을 나눔으로써 환경에 따라 다른 로그를 찍을 수도 있습니다.**

### **2-4. 로그 레벨**

**총 5가지의 레벨이 있습니다. 왼쪽이 가장 낮은 레벨이고, 오른쪽이 가장 높은 레벨입니다.**

**TRACE < DEBUG  < INFO < WARN  < ERROR**

**1) ERROR : 요청을 처리하는 중 오류가 발생한 경우 표시한다.**

**2) WARN : 처리 가능한 문제, 향후 시스템 에러의 원인이 될 수 있는 경고성 메시지를 나타낸다.**

**3) INFO : 상태 변경과 같은 정보성 로그를 표시한다.**

**4) DEBUG : 프로그램을 디버깅하기 위한 정보를 표시한다.**

**5) TRACE : 추적 레벨은 Debug보다 훨씬 상세한 정보를 나타낸다.**

**만약 log 레벨을 INFO로 설정했다면, INFO부터 ~ERROR까지 자신보다 높은 로그까지 찍게 됩니다.**

**ex) DEBUG로 설정할 시 DEBUG~ERROR까지 로그를 남기고, TRACE는 로그를 남기지 않습니다.**

```
logging:  level:    org.hibernate.SQL: debug
```

### **2-5. 로그 범위 지정**

**위에서 살펴본 로그는 하이버네이트가 SQL을 실행할 때 DEBUG보다 높은 로그들을 남기는 것입니다. 따라서 전체 혹은 패키지 별로 로그를 남기도록 할 수 있습니다. 아래와 같이 전체 부분에 대해서 warn위는 반드시 로그를 남기도록 할 수 있고, com.example.log에 해당하는 패키지는 debug부터 남길 수 있도록 설정할 수 있습니다.**

```
logging:  level:    root: warn    com.example.log: debug
```

## **3. application.yml로 로그 적용**

**위에서 잠시 살펴본 것이 application.yml로 log를 적용한 것입니다. 각각 설정에 대해서 알아보겠습니다. 각 설명은 주석으로 남기겠습니다.**

```yaml
logging:
  level: # 적용할 곳과 레벨을 지정합니다.
    root: warn
    com.example.log: debug
  file: # 로그 파일에 대한 명시입니다. 주의점 name과 path중 하나만 사용해야합니다. path 사용을 권장합니다.
    name: # 로그 파일의 이름
    path: # 로그 파일의 위치 절대 경로를 적어주시면 됩니다.
  logback:
    rollingpolicy:
      file-name-pattern: # 로그 파일의 이름을 설정하는 패턴입니다. 파일명+날짜와 같이 지정할 수 있습니다.
      clean-history-on-start: # 애플리케이션 재실행시 로그 파일을 초기화 여부 설정
      max-file-size: # 로그 파일의 최대 크기이며, 크기가 넘어가면 새로운 로그파일 작성
      max-history: # 로그 파일의 최대 수
      total-size-cap: # 로그 파일의 총 크기이며, 넘어갈 시 가장 오래된 로그 파일 삭제
  pattern:
    console: # 콘솔 창에 출력될 로그의 패턴
    level: # 출력 로그 레벨 지정
    file: # 로그 파일에 사용될 로그 패턴
    dateformat: # 로그의 date에 대한 디폴트 설정
```

## **4. logback-spring.xml 로그 적용**

**logback-spring.xml 참조: [https://goddaehee.tistory.com/206](https://goddaehee.tistory.com/206)**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- 60초마다 설정 파일의 변경을 확인 하여 변경시 갱신 -->
<configuration scan="true" scanPeriod="60 seconds">
    <!--springProfile 태그를 사용하면 logback 설정파일에서 복수개의 프로파일을 설정할 수 있다.-->
    <springProfile name="local">
        <property resource="logback-local.properties"/>
    </springProfile>
    <springProfile name="dev">
        <property resource="logback-dev.properties"/>
    </springProfile>
    <!--Environment 내의 프로퍼티들을 개별적으로 설정할 수도 있다.-->
    <springProperty scope="context" name="LOG_LEVEL" source="logging.level.root"/>
 
    <!-- log file path -->
    <property name="LOG_PATH" value="${log.config.path}"/>
    <!-- log file name -->
    <property name="LOG_FILE_NAME" value="${log.config.filename}"/>
    <!-- err log file name -->
    <property name="ERR_LOG_FILE_NAME" value="err_log"/>
    <!-- pattern -->
    <property name="LOG_PATTERN" value="%-5level %d{yy-MM-dd HH:mm:ss}[%thread] [%logger{0}:%line] - %msg%n"/>
 
    <!-- Console Appender -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>${LOG_PATTERN}</pattern>
        </encoder>
    </appender>
 
    <!-- File Appender -->
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 파일경로 설정 -->
        <file>${LOG_PATH}/${LOG_FILE_NAME}.log</file>
 
        <!-- 출력패턴 설정-->
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>${LOG_PATTERN}</pattern>
        </encoder>
 
        <!-- Rolling 정책 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- .gz,.zip 등을 넣으면 자동 일자별 로그파일 압축 -->
            <fileNamePattern>${LOG_PATH}/${LOG_FILE_NAME}.%d{yyyy-MM-dd}_%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <!-- 파일당 최고 용량 kb, mb, gb -->
                <maxFileSize>10MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <!-- 일자별 로그파일 최대 보관주기(~일), 해당 설정일 이상된 파일은 자동으로 제거-->
            <maxHistory>30</maxHistory>
            <!--<MinIndex>1</MinIndex>
            <MaxIndex>10</MaxIndex>-->
        </rollingPolicy>
    </appender>
 
    <!-- 에러의 경우 파일에 로그 처리 -->
    <appender name="Error" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>error</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
        <file>${LOG_PATH}/${ERR_LOG_FILE_NAME}.log</file>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>${LOG_PATTERN}</pattern>
        </encoder>
        <!-- Rolling 정책 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- .gz,.zip 등을 넣으면 자동 일자별 로그파일 압축 -->
            <fileNamePattern>${LOG_PATH}/${ERR_LOG_FILE_NAME}.%d{yyyy-MM-dd}_%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <!-- 파일당 최고 용량 kb, mb, gb -->
                <maxFileSize>10MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <!-- 일자별 로그파일 최대 보관주기(~일), 해당 설정일 이상된 파일은 자동으로 제거-->
            <maxHistory>60</maxHistory>
        </rollingPolicy>
    </appender>
 
    <!-- root레벨 설정 -->
    <root level="${LOG_LEVEL}">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="FILE"/>
        <appender-ref ref="Error"/>
    </root>
 
    <!-- 특정패키지 로깅레벨 설정 -->
    <logger name="org.apache.ibatis" level="DEBUG" additivity="false">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="FILE"/>
        <appender-ref ref="Error"/>
    </logger>
</configuration>
```

## **5. yml과 logback-spring.xml비교**

**logback-spring.xml과 다르게 yml로 작성 시 매우 간단하고 쉽게 작성할 수 있습니다. 하지만 문제점이 있습니다. yml로 작성 시 로그 레벨별로 같은 세부 적인 설정 못하는 단점이 있습니다. 간단한 설정은 yml로 작성하고, 로그 레벨별로 나누는 것과 같은 설정은 logback-spring.xml로 작성하면 되겠습니다.**