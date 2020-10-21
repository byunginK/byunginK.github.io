---
layout: post
title:  "JAVA JDBC 연동"
date:   2020-06-20
categories: [java]
---

# JDBC Connection
## 이클립스 와  DB 연동
1. 이클립스에 DataBase development에 오라클의 주소및 서버가 연동을 시킨것을 기반으로 자바 프로젝트와 연동을 설명
2. JAVA 프로젝트에서 우클릭하면 build path 부분이 있다 configure build path... 의 부분을 들어간다.
3. Libraries 탭으로 들어가서 add external jar을 클릭하여 추가하여준다.

- 기본적인 setting이 끝났다.

## 오라클 드라이브와 연동
- 가장 먼저 연동해야 할 부분은 드라이브 연동이다.
- 드라이브 연동과 DB 서버 연동은 따로 패키지를 만들고 메소드로 진행한다. 
- 또한 static을 설정하여 어디서든지 접근할 수 있게 하면 JDBC 할 수 있다.

```java
public static void initConnection() {                   // static 을 붙이는 이유는 어디서든지 바로 접근이 가능하게 끔 한다.
		try {
			Class.forName("oracle.jdbc.driver.OracleDriver"); //  Class.forName 은 드라이버가 있는지 확인하는 식

			System.out.println("Driver Loading Success");

		} catch (ClassNotFoundException e) {

			e.printStackTrace();
		}
	}
```
## DB 서버와 연동
```java
public static Connection getConnection() {     // Connection (java.sql) = jdbc의 오브젝트를 읽어준다.
		Connection conn = null;
		
		try {
			conn = DriverManager.getConnection("jdbc:oracle:thin:@192.168.7.45:1521:xe", "hr", "hr");  // url 은 연동된 DB의 url  프로퍼티에서 복붙 그리고 사용자, 비번 입력
			
			System.out.println("Oracle Connection Success");
			
		} catch (SQLException e) {
			System.out.println("Oracle Connection Fail");
		}
		return conn;  // return 값으로 연결한 인스턴스를 반환한다.
	}
```

## DBConnection 객체
```java
package db;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class DBConnection {

	public static void initConnection() {  
		try {
			Class.forName("oracle.jdbc.driver.OracleDriver"); 

			System.out.println("Driver Loading Success");

		} catch (ClassNotFoundException e) {

			e.printStackTrace();
		}
	}
	
	public static Connection getConnection() { 
		Connection conn = null;
		
		try {
			conn = DriverManager.getConnection("jdbc:oracle:thin:@192.168.7.45:1521:xe", "hr", "hr"); 
			
			System.out.println("Oracle Connection Success");
			
		} catch (SQLException e) {
			System.out.println("Oracle Connection Fail");
		}
		return conn;  
	}
}
```
**※ DB 서버는 값을 주고 닫아주어야 한다. 그렇지 않으면 서버에 대한 문제가 발생할 수 있다. 위에 생성한 db 패키지에 close 클래스도 만들어 주고 활용한다.**  

```java
package db;

import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

public class DBClose {   // 해방시키려는 부분은 클래스로 따로 생성하는 편
	
	public static void close(Statement stmt, Connection conn, ResultSet rs) {
		
		try {
			if(stmt != null) {
				stmt.close();
			}	
			if(conn != null) {
				conn.close();
			}
			if(rs != null) {   // Select 절일 때 필요한 클래스
				rs.close();
			}
		} catch (SQLException e) {
			e.printStackTrace();
		}
		
	}
}
```

# JDBC Select
## Dto 생성
- 데이터에 저장 되어있는 값을 받아와 명시하기 위해서 dto를 생성해주어야 한다.
- 현재 예제에는 상관이 없지만 Web에서 DB와 java를 연동해서 사용할 경우 dto에 직렬화를 해주어야한다.<br>
(해당 부분은 네트워크의 TCP 에서 객체를 송/수신할때 한번 언급하였다.)
```java
package dto;

import java.io.Serializable;

public class UserDto implements Serializable {  // 직렬화를 해주어야 한다. 서버로 통해서 데이터를 오고 가기때문데 (웹에서)
	
	private String id;
	private String name;
	private int age;
	private String Joindate;
	
	public UserDto() {
	}
	
	public UserDto(String id, String name, int age, String joindate) {
		super();
		this.id = id;
		this.name = name;
		this.age = age;
		Joindate = joindate;
	}

	public String getId() {
		return id;
	}

	public void setId(String id) {
		this.id = id;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public int getAge() {
		return age;
	}

	public void setAge(int age) {
		this.age = age;
	}

	public String getJoindate() {
		return Joindate;
	}

	public void setJoindate(String joindate) {
		Joindate = joindate;
	}

	@Override
	public String toString() {
		return "UserDto [id=" + id + ", name=" + name + ", age=" + age + ", Joindate=" + Joindate + "]";
	}
	
}
```
## Select Class
### 1. 단일 데이터를 검색할때
- statement 사용
```java
public UserDto search(String id) {
		
		String sql = " SELECT ID, NAME, AGE, JOINDATE "
				+ " FROM USERTEST "
				+ " WHERE ID = '"+ id + "' ";
		
		System.out.println("sql: "+ sql);
		
		Connection conn = DBConnection.getConnection();
		Statement stmt = null;
		
		ResultSet rs = null;    // DB로 부터 결과를 return
		
		UserDto dto = null;
		
		try {
			stmt = conn.createStatement();
			
			rs = stmt.executeQuery(sql);
			
			if(rs.next()) {                      // 찾는 데이터가 존재 하는 경우 true  없으면 false
				String _id = rs.getString("id");   // getString(" DB 생성시 만들었던 컬럼명을 넣어준다.")
				String _name = rs.getString("name"); 
				int _age = rs.getInt("age");
				String _joindate = rs.getString("joindate");
				
				dto = new UserDto(_id, _name, _age, _joindate); // dto에 받은 값을 넣어주고 객체를 생성  
			}
			// 만약 값이 없어서 그냥 넘어오게 되면 값은 null이다.
			
		} catch (SQLException e) {
			e.printStackTrace();
		} finally {
			DBClose.close(stmt, conn, rs);
		}
		return dto;
	}
  ```
  - PreparedStatement
  ```java
  public UserDto select(String id) {
		
		String sql = " SELECT ID, NAME, AGE, JOINDATE "
					+ " FROM USERTEST "
					+ " WHERE ID = ? AND AGE = ? "; // 예를 들기위해 두가지 조건을 걸었다.
		
		Connection conn = DBConnection.getConnection();
		PreparedStatement psmt = null;
		ResultSet rs = null;
		
		UserDto dto = null;
		try {
			psmt = conn.prepareStatement(sql);
			psmt.setString(1, id);   //  Query에서  ? 부분에 해당. ''의 생략이 가능하다 . 자동적으로 붙는다. 이 시점에서 Query 문이 완성된다.
			psmt.setInt(2, 25); // 두번째 age의 조건이며 age컬럼은 2번째이고, 값은 직접 넣어주어도 되고, 파라미터로 id처럼 받아서 값을 넣어도 된다.
			
			rs = psmt.executeQuery();
			
			if(rs.next()) {
				dto = new UserDto();
				
				dto.setId(rs.getString("id"));   // 
				dto.setName(rs.getString("name"));
				dto.setAge(rs.getInt("age"));
				dto.setJoindate(rs.getString("joindate"));
			}
			
			
		} catch (SQLException e) {
			e.printStackTrace();
		} finally {
			DBClose.close(psmt, conn, rs);
		}
		return dto;
	}
  ```
  
  ### 2. 다중 데이터를 검색
  ```java
  public List<UserDto> getUserList() {
		
		String sql = " SELECT ID, NAME, AGE, JOINDATE "
				+ " FROM USERTEST ";
		
		Connection conn = DBConnection.getConnection();
		PreparedStatement psmt = null;
		ResultSet rs = null;
		
		List<UserDto> list = new ArrayList<UserDto>();
		
		try {
			psmt = conn.prepareStatement(sql);
			rs = psmt.executeQuery();
			
			while(rs.next()) {     //  데이터가 있을 경우 true 이기 때문에 계속 돌아가면서 list에 값을 넣어줄 수 있다.
				String id = rs.getString("id");
				String name = rs.getString("name");
				int age = rs.getInt("age");
				String joindate = rs.getString("joindate");
				
				UserDto dto = new UserDto(id, name, age, joindate);
				
				list.add(dto); 
						
			}
			
		} catch (SQLException e) {
			e.printStackTrace();
		} finally {
			DBClose.close(psmt, conn, rs);
		}
		return list;
	}
  ```
  
  ## Main
  ```java
  import java.util.List;

import db.DBConnection;
import dto.UserDto;
import jdbc.SelectTest;

public class Main {

	public static void main(String[] args) {
		
		
		DBConnection.initConnection();
		
		SelectTest st = new SelectTest();   // statement 를 사용한 select
		
		String id ="abc";
		UserDto dto = st.search(id);
		if( dto != null) {
			System.out.println(dto.toString());
		}
		else {
			System.out.println("등록되어 있지 않은 ID 입니다.");
		}
		
		id = "abc3";               // preparedStatement 를 사용한 select
		dto= st.select(id);
		if( dto != null) {
			System.out.println(dto.toString());
		}
		else {
			System.out.println("등록되어 있지 않은 ID 입니다.");
		}
		
		List<UserDto> list = st.getUserList();    // 다중 데이터를 select
		for (UserDto user : list) {
			System.out.println(user.toString());
		}
	}
}
```

# JDBC Insert
## InsertData Method 
```java
public int insertData(String id, String name, int age) { // insert, update, delete는 넘어오는 값이 정수(= count)로 넘어온다.
 		
		// Query -> String 문자열로 되어있다.
    
		String sql = " INSERT INTO USERTEST(ID, NAME, AGE, JOINDATE) "
				+ " VALUES('" + id + "','"+ name + "',"+ age+", SYSDATE)";   // 매개변수 넣을 때 " 안에 '를 넣어주어야한다.
				
 		System.out.println("sql: "+ sql);
 		
 		Connection conn = DBConnection.getConnection();
 		Statement stat = null;                           // 현재 상태를 확인하는 단계
 		int count = 0;  // return 을 해주기 위해서  (몇개가 INSERT되었는지 확인)
		
 		try {
			stat = conn.createStatement();
			
			count = stat.executeUpdate(sql);  // executeupdate 는 return 값이 int 이기 때문에 count에 업데이트된 수를 넣어준다.
			
			System.out.println("성공적으로 추가 되었습니다.");
			
		} catch (SQLException e) {        // DB가 꺼져있거나 연동 안되어있을경우 예외
			e.printStackTrace();
		}finally {
    
			// DB를 사용하고 반드시 닫아주어야 한다. 그래서 finally를 통해 무조건 닫는다.
      
			DBClose.close(stat, conn, null);  // ResultSet에 대한 부분은 insert에서 초기화하기 않기 때문에 null로 값을 넣어준다.
		}
 		return count;
	}
  ```
  
  ## Inesrt Class
  ```java
  package jdbc;

import java.sql.Connection;
import java.sql.SQLException;
import java.sql.Statement;

import db.DBClose;
import db.DBConnection;

public class InsertTest {

	public InsertTest() {
	}
	public int insertData(String id, String name, int age) { 
 		
		String sql = " INSERT INTO USERTEST(ID, NAME, AGE, JOINDATE) "
				+ " VALUES('" + id + "','"+ name + "',"+ age+", SYSDATE)"; 
				
 		System.out.println("sql: "+ sql);
 		
 		Connection conn = DBConnection.getConnection();
 		Statement stat = null;                           
 		int count = 0;  
		
 		try {
			stat = conn.createStatement();
			
			count = stat.executeUpdate(sql);  
			
			System.out.println("성공적으로 추가 되었습니다.");
			
		} catch (SQLException e) { 
			e.printStackTrace();
		}finally {
			DBClose.close(stat, conn, null);
		}
 		return count;
	}
}
```

## Main

```java
package main;

import db.DBConnection;
import jdbc.InsertTest;

public class Main {

	public static void main(String[] args) {
		
		DBConnection.initConnection(); // 생성자에 굳이 넣을 필요없이 바로 불러오기만 하면된다.
		
		InsertTest it = new InsertTest(); 
		int count = it.insertData("abc4", "장영실", 22); // 데이터 insert
		if(count == 1) {
			System.out.println("데이터가"+ count + "개 추가 되었습니다.");
		}
	}
}
```
# JDBC Delete
## Delete Class
```java
package jdbc;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.SQLException;

import db.DBClose;
import db.DBConnection;

public class DeleteTest {

	public boolean deleteTest(String id) {
		
		String sql = " DELETE FROM USERTEST "
				+ " WHERE ID = ? ";       // 이전 insert 와 update에서 사용한 ' ' 가 아닌 ? 를 사용할 수 있다.
		
		System.out.println("sql: "+sql);
		
		Connection conn = DBConnection.getConnection();
		PreparedStatement psmt = null;  // WHERE 절에서 ? 를 사용할시 preparedStatement를 사용해야한다.
		
		int count = 0;
		
		try {
			psmt = conn.prepareStatement(sql);   // 쿼리문을 넣어주고 여기서 쿼리문이 완성된다.
			psmt.setString(1, id);               // ? 부분의 컬럼과 들어갈 값이 무엇인지 set 해준다. (1번째 컬럼(ID), id (String id)파라미터로 받은 값을 대입)
			count = psmt.executeUpdate();  
			
			
			
		} catch (SQLException e) {
			e.printStackTrace();
		} finally {
			DBClose.close(psmt, conn, null);
		}
		return count > 0? true:false;
	}
}
```

## Main
```java
package main;

import db.DBConnection;
import jdbc.DeleteTest;

public class Main {

	public static void main(String[] args) {
		DBConnection.initConnection();
		
		DeleteTest dt = new DeleteTest();
		
		String id = "abc4";   // 삭제할 ID 
		boolean b = dt.deleteTest(id);
		
		if(b == true) {
			System.out.println("정상적으로 삭제되었습니다.");
		}
	}
}
```
# JDBC Update
## Update Class
- Inesrt와 비슷한 방식으로 구현되며 쿼리문만 update로 바뀐다.
- 메소드를 boolean형식으로 return되도록 하였다. 이유는 메인 클래스에서 확인

```java
package jdbc;

import java.sql.Connection;
import java.sql.SQLException;
import java.sql.Statement;

import db.DBClose;
import db.DBConnection;

public class UpdateTest {

	public boolean update(String id, int age) {
		
		String sql = " UPDATE USERTEST "
				   + " SET AGE = " + age + " "
				   + " WHERE ID = '"+ id + "' ";
		System.out.println("sql : "+sql);
		
		Connection conn = DBConnection.getConnection();
		Statement stmt = null;          // 쿼리문을 실행시켜서 결과를 가져오는 목적을 가진 소스코드
		
		int count = 0;
		
		try {
			stmt = conn.createStatement();
			count = stmt.executeUpdate(sql);
			
			
		} catch (SQLException e) {
			e.printStackTrace();
		} finally {
			DBClose.close(stmt, conn, null);
		}
		
		return count > 0 ? true:false; //  삼항연산자로 리턴값 반환가능 , count가 0 이상이면 stmt가 1개이상 값을 count로 주었기 때문에  true가 나옴
		
	}
}
```
## Main
- 우선 update할 id를 지정하고 변경할 age 값도 지정한 후 update 파라미터로 넣어준다.
- return값은 True, false 이기 때문에 boolean b 를 잡아주고 조건절을 통해 정상적으로 수정 되었는지 확인 할 수 있다.
```java
package main;

import db.DBConnection;
import jdbc.UpdateTest;

public class Main {

	public static void main(String[] args) {

		DBConnection.initConnection(); // DB 동작 준비 끝
		
		UpdateTest ut = new UpdateTest();
		
		String id = "abc3";
		
		int age = 45;
		
		boolean b = ut.update(id, age);
		if(b == true) {
			System.out.println("정상적으로 수정 되었습니다.");
		}
	}
}
```

