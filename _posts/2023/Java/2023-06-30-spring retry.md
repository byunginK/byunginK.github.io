---
layout: post
title: "Spring Retry 기능"
date: 2023-06-30
categories: [Spring]
---
# spring boot retry

# **2. Maven Dependencies**

Let's begin by **adding the *spring-retry* dependency into our *pom.xml* file**:

```java
<dependency>
    <groupId>org.springframework.retry</groupId>
    <artifactId>spring-retry</artifactId>
    <version>2.0.0</version>
</dependency>Copy
```

We also need to add Spring AOP in our project:

```java
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aspects</artifactId>
    <version>5.2.8.RELEASE</version>
</dependency>Copy
```


# **3. Enabling Spring Retry**

To enable Spring Retry in an application, **we need to add the *@EnableRetry* annotation** to our *@Configuration* class:

```java
@Configuration
@EnableRetry
publicclassAppConfig { ... }
```

# **4. Using Spring Retry**

### **4.1. *@Retryable* Without Recovery**

**We can use the *@Retryable* annotation to add retry functionality to methods**:

```java
@Service
publicinterfaceMyService {

    @Retryable
voidretryService(String sql);
}
```

Since we have not specified any exceptions here, retry will be attempted for all the exceptions. Also, once the max attempts are reached and there is still an exception, ExhaustedRetryException will be thrown.

Per *@Retryable*‘s default behavior, **the retry may happen up to three times, with a delay of one second between retries.**

### **4.2. *@Retryable* and *@Recover***

**Let's now add a recovery method using the *@Recover* annotation**:

```java
@Service
publicinterfaceMyService {

    @Retryable(retryFor = SQLException.class)
voidretryServiceWithRecovery(String sql)throws SQLException;

    @Recover
voidrecover(SQLException e, String sql);
}
```

Here, the retry is attempted when an *SQLException* is thrown*.* **The *@Recover* annotation defines a separate recovery method when a *@Retryable* method fails with a specified exception.**

Consequently, if the *retryServiceWithRecovery* method keeps throwing an *SqlException* after three attempts, the *recover()* method will be called.

The recovery handler should have the first parameter of type *Throwable* (optional) and the same return type. **The following arguments are populated from the argument list of the failed method in the same order.

### **4.3. Customizing *@Retryable's* Behavior**

In order to customize a retry's behavior, **we can use the parameters *maxAttempts* and *backoff***:

```java
@Service
publicinterfaceMyService {

    @Retryable(retryFor = SQLException.class, maxAttempts = 2, backoff = @Backoff(delay = 100))
voidretryServiceWithCustomization(String sql)throws SQLException;
}
```

There will be up to two attempts and a delay of 100 milliseconds.

### **4.4. Using Spring Properties**

We can also use properties in the *@Retryable* annotation.

To demonstrate this, **we'll see how to externalize the values of *delay* and *max attempts* into a properties file.**

First, let's define the properties in a file called *retryConfig.properties*:

```
retry.maxAttempts=2
retry.maxDelay=100Copy
```

We then instruct our *@Configuration* class to load this file:

```java
// ...
@PropertySource("classpath:retryConfig.properties")
publicclassAppConfig { ... }
```

Finally, **we can inject the values of *retry.maxAttempts* and *retry.maxDelay* in our *@Retryable* definition**:

```java
@Service
publicinterfaceMyService {

    @Retryable(retryFor = SQLException.class, maxAttemptsExpression = "${retry.maxAttempts}",
               backoff = @Backoff(delayExpression = "${retry.maxDelay}"))
voidretryServiceWithExternalConfiguration(String sql)throws SQLException;
}
```

Please note that **we are now using *maxAttemptsExpression* and *delayExpression*** instead of *maxAttempts* and *delay*.

# **5. *RetryTemplate***

### **5.1. *RetryOperations***

Spring Retry provides *RetryOperations* interface, which supplies a set of *execute()* methods:

```java
publicinterfaceRetryOperations {

    <T> Texecute(RetryCallback<T> retryCallback)throws Exception;

    ...
}
```

The *RetryCallback*, which is a parameter of the *execute()*, is an interface that allows the insertion of business logic that needs to be retried upon failure:

```java
publicinterfaceRetryCallback<T> {

    TdoWithRetry(RetryContext context)throws Throwable;
}
```

### **5.2. *RetryTemplate* Configuration**

The *RetryTemplate* is an implementation of the *RetryOperations*.

Let's configure a *RetryTemplate* bean in our *@Configuration* class:

```java
@Configuration
publicclassAppConfig {

    //...

    @Bean
public RetryTemplateretryTemplate() {
RetryTemplate retryTemplate =newRetryTemplate();

FixedBackOffPolicy fixedBackOffPolicy =newFixedBackOffPolicy();
        fixedBackOffPolicy.setBackOffPeriod(2000l);
        retryTemplate.setBackOffPolicy(fixedBackOffPolicy);

SimpleRetryPolicy retryPolicy =newSimpleRetryPolicy();
        retryPolicy.setMaxAttempts(2);
        retryTemplate.setRetryPolicy(retryPolicy);

return retryTemplate;
    }
}

```

The *RetryPolicy* determines when an operation should be retried.

A *SimpleRetryPolicy* is used to retry a fixed number of times. On the other hand, the *BackOffPolicy* is used to control backoff between retry attempts.

Finally, a *FixedBackOffPolicy* pauses for a fixed period of time before continuing.

### **5.3. Using the *RetryTemplate***

To run code with retry handling, we can call the *retryTemplate.execute()* method:

```java
retryTemplate.execute(newRetryCallback<Void, RuntimeException>() {
    @Override
public VoiddoWithRetry(RetryContext arg0) {
        myService.templateRetryService();
        ...
    }
});
```

Instead of an anonymous class, we can use a lambda expression:

```java
retryTemplate.execute(arg0 -> {
    myService.templateRetryService();
return null;
});
```

# **6. Listeners**

Listeners provide additional callbacks upon retries. And we can use these for various cross-cutting concerns across different retries.

### **6.1. Adding Callbacks**

The callbacks are provided in a *RetryListener* interface:

```java
publicclassDefaultListenerSupportextendsRetryListenerSupport {

    @Override
public <T, EextendsThrowable>voidclose(RetryContext context,
      RetryCallback<T, E> callback, Throwable throwable) {
        logger.info("onClose");
        ...
        super.close(context, callback, throwable);
    }

    @Override
public <T, EextendsThrowable>voidonError(RetryContext context,
      RetryCallback<T, E> callback, Throwable throwable) {
        logger.info("onError");
        ...
        super.onError(context, callback, throwable);
    }

    @Override
public <T, EextendsThrowable>booleanopen(RetryContext context,
      RetryCallback<T, E> callback) {
        logger.info("onOpen");
        ...
return super.open(context, callback);
    }
}
```

The *open* and *close* callbacks come before and after the entire retry, while *onError* applies to the individual *RetryCallback* calls.

### **6.2. Registering the Listener**

Next, we register our listener (*DefaultListenerSupport)* to our *RetryTemplate* bean:

```java
@Configuration
publicclassAppConfig {
    ...

    @Bean
public RetryTemplateretryTemplate() {
RetryTemplate retryTemplate =newRetryTemplate();
        ...
        retryTemplate.registerListener(newDefaultListenerSupport());
return retryTemplate;
    }
}
```

# **7. Testing the Results**

To finish our example, let's verify the results:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(
  classes = AppConfig.class,
  loader = AnnotationConfigContextLoader.class)
publicclassSpringRetryIntegrationTest {

    @Autowired
private MyService myService;

    @Autowired
private RetryTemplate retryTemplate;

    @Test(expected = RuntimeException.class)
publicvoidgivenTemplateRetryService_whenCallWithException_thenRetry() {
        retryTemplate.execute(arg0 -> {
            myService.templateRetryService();
return null;
        });
    }
}
```

As we can see from the test logs, we have properly configured the *RetryTemplate* and the *RetryListener*:

```
2020-01-09 20:04:10 [main] INFO  o.b.s.DefaultListenerSupport - onOpen
2020-01-09 20:04:10 [main] INFO  o.baeldung.springretry.MyServiceImpl
- throw RuntimeException in method templateRetryService()
2020-01-09 20:04:10 [main] INFO  o.b.s.DefaultListenerSupport - onError
2020-01-09 20:04:12 [main] INFO  o.baeldung.springretry.MyServiceImpl
- throw RuntimeException in method templateRetryService()
2020-01-09 20:04:12 [main] INFO  o.b.s.DefaultListenerSupport - onError
2020-01-09 20:04:12 [main] INFO  o.b.s.DefaultListenerSupport - onClose
```