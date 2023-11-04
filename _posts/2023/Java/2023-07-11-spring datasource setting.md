---
layout: post
title: "Spring DataSource 세팅"
date: 2023-07-11
categories: [Spring]
---
# Configuring a DataSource Programmatically in Spring Boot

[Spring Boot](https://www.baeldung.com/spring-boot) uses an opinionated algorithm to scan for and configure a *[DataSource](https://docs.oracle.com/en/java/javase/17/docs/api/java.sql/javax/sql/DataSource.html)*. This allows us to easily get a fully-configured *DataSource* implementation by default.

In addition, Spring Boot automatically configures a lightning-fast [connection pool](https://www.baeldung.com/java-connection-pooling), either [HikariCP](https://www.baeldung.com/hikaricp), [Apache Tomcat](https://www.baeldung.com/spring-boot-tomcat-connection-pool), or [Commons DBCP](https://commons.apache.org/proper/commons-dbcp/), in that order, depending on which are on the classpath.

**While Spring Boot's automatic *DataSource* configuration works very well in most cases, sometimes we'll need a higher level of control**, so we'll have to set up our own *DataSource* implementation, hence skipping the automatic configuration process.

In this tutorial, we'll learn **how to configure a *DataSource* programmatically in Spring Boot**.

## Further reading:

## [Spring JPA – Multiple Databases](https://www.baeldung.com/spring-data-jpa-multiple-databases)

How to set up Spring Data JPA to work with multiple, separate databases.

## [Configuring Separate Spring DataSource for Tests](https://www.baeldung.com/spring-testing-separate-data-source)

A quick, practical tutorial on how to configure a separate data source for testing in a Spring application.

## 2. The Maven Dependencies

**Creating a *DataSource* implementation programmatically is straightforward overall**.

To learn how to accomplish this, we'll implement a simple repository layer, which will perform CRUD operations on some [JPA](https://www.baeldung.com/the-persistence-layer-with-spring-and-jpa) entities.

Let's take a look at our demo project's dependencies:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>2.4.1</version>
    <scope>runtime</scope>
</dependency>
```

As shown above, we'll use an in-memory [H2 database](https://search.maven.org/search?q=g:org.hsqldb%20AND%20a:hsqldb) instance to exercise the repository layer. In doing so, we'll be able to test our programmatically-configured *DataSource,* without the cost of performing expensive database operations.

In addition, let's make sure to check the latest version of *[spring-boot-starter-data-jpa](https://search.maven.org/search?q=g:org.springframework.boot%20AND%20a:spring-boot-starter-data-jpa)* on Maven Central.

## **3. Configuring a *DataSource* Programmatically**

Now, if we stick with Spring Boot's automatic *DataSource* configuration and run our project in its current state, it will work just as expected.

**Spring Boot will do all the heavy infrastructure plumbing for us.** This includes creating an H2 *DataSource* implementation, which will be automatically handled by HikariCP, Apache Tomcat, or Commons DBCP, and setting up an in-memory database instance.

Additionally, we won't even need to create an *application.properties* file, as Spring Boot will provide some default database settings as well.

As we mentioned before, at times we'll need a higher level of customization, so we'll have to programmatically configure our own *DataSource* implementation.

**The simplest way to accomplish this is by defining a *DataSource* factory method, and placing it within a class annotated with the *[@Configuration](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Configuration.html)* annotation**:

```java
@Configuration
public class DataSourceConfig {

    @Bean
    public DataSource getDataSource() {
        DataSourceBuilder dataSourceBuilder = DataSourceBuilder.create();
        dataSourceBuilder.driverClassName("org.h2.Driver");
        dataSourceBuilder.url("jdbc:h2:mem:test");
        dataSourceBuilder.username("SA");
        dataSourceBuilder.password("");
        return dataSourceBuilder.build();
    }
}
```

In this case, **we used the convenient *[DataSourceBuilder](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/jdbc/DataSourceBuilder.html)* class,** a non-fluent version of [Joshua Bloch's builder pattern](https://www.pearson.com/us/higher-education/program/Bloch-Effective-Java-3rd-Edition/PGM1763855.html), **to programmatically create our custom *DataSource* object**.

## **6. Conclusion**

In this article, **we learned how to configure a *DataSource* implementation programmatically in Spring Boot**.

As usual, all the code samples shown in this article are available [over on GitHub](https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-boot-persistence).