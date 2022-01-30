---
layout: post
title:  "Web Spring boot Mybatis 데이터 받아오기 에러"
date:   2020-10-23
categories: [web]
---

# DB 데이터 불러올때 Error

###### Project를 Spring boot를 활용하여 진행 하였다. DB 연동을 MyBais를 이용하여 쿼리문을 작성 하였는데, 문제가 발생하였다.

###### Select 쿼리문에서 일부 데이터만 정상적으로 받아와 지고 일부 데이터는 null 값으로 데이터를 불러 왔다.

###### 원인을 찾기위해 여러가지 방법을 시도 하던 중  DB Config 부부을 살펴 보다가 문제를 발견 하였다.

###  DB Config Code
```java
@Configuration
/* @MapperScan(basePackages="com.bit.paperhouse.dao") */
@EnableTransactionManagement
public class DatabaseConfig {
	
	//데이터베이스 경로설정
	@Bean
	public SqlSessionFactory sqlSessionFactory( DataSource dataSource ) throws Exception {
		System.out.println("DatabaseConfig  SqlSessionFactory");
		
		SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
		sqlSessionFactoryBean.setDataSource(dataSource);
		
		Resource[] arrResource = new PathMatchingResourcePatternResolver().getResources("classpath:mapper/*.xml");
		sqlSessionFactoryBean.setMapperLocations(arrResource);
		sqlSessionFactoryBean.getObject().getConfiguration().setMapUnderscoreToCamelCase(true);
		
		return (SqlSessionFactory)sqlSessionFactoryBean.getObject();
	}
	
	@Bean
	public SqlSessionTemplate sqlSession( SqlSessionFactory sqlSessionFactory) {
		
		return new SqlSessionTemplate(sqlSessionFactory);
		
	}

}
```
###### 위 코드 에서 `sqlSessionFactoryBean.getObject().getConfiguration().setMapUnderscoreToCamelCase(true);` 코드가 원인이였다. 

###### SqlSessionFactoryBean의 CamelCase만 인식을 한다고 설정이 되어있었다.  Mybatis 쿼리문과 result값을 받는 Dto의 변수들을 모두 CamelCase로 변경 후 다시 Select을 하였더니 정상적으로 값들을 받아 왔다.
