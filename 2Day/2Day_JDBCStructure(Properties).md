<br>

## common.JDBCTemplate
<br>

```java
package common;

import java.io.FileReader;
import java.io.IOException;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.Properties;

public class JDBCTemplate {
	public static Connection getConnection() {
		Connection con = null;

		try {
			Properties prop = new Properties();
			prop.load(new FileReader("driver.properties"));

			Class.forName(prop.getProperty("driver"));
			con = DriverManager.getConnection(prop.getProperty("url"), prop.getProperty("user"),prop.getProperty("password"));

			con.setAutoCommit(false);
		} catch (IOException e) {
			e.printStackTrace();
		} catch (ClassNotFoundException e) {
			e.printStackTrace();
		} catch (SQLException e) {
			e.printStackTrace();
		}

		return con;
	}

	public static void commit(Connection con) {
		try {
			if (con != null && !con.isClosed()) {
				con.commit();
			}
		} catch (SQLException e) {
			e.printStackTrace();
		}
	}

	public static void rollback(Connection con) {
		try {
			if (con != null && !con.isClosed()) {
				con.rollback();
			}
		} catch (SQLException e) {
			e.printStackTrace();
		}
	}

	public static void close(Connection con) {
		try {
			if (con != null && !con.isClosed()) {
				con.close();
			}
		} catch (SQLException e) {
			e.printStackTrace();
		}
	}

	public static void close(Statement stmt) {
		try {
			if (stmt != null && !stmt.isClosed()) {
				stmt.close();
			}
		} catch (SQLException e) {
			e.printStackTrace();
		}
	}

	public static void close(ResultSet rset) {
		try {
			if (rset != null && !rset.isClosed()) {
				rset.close();
			}
		} catch (SQLException e) {
			e.printStackTrace();
		}
	}

}
```

<br>
<hr>
<br>

## run.TestMain
<br>

```java
package run;

import test.view.TestMenu;

public class TestMain {
	public static void main(String[] args) {
		TestMenu tm = new TestMenu();
		tm.displayMenu();
	}
}
```

<br>
<hr>
<br>

## test.controller.TestController

<br>

```java
package test.controller;

import java.util.ArrayList;

import test.model.service.TestService;
import test.model.vo.Test;
import test.view.TestView;

public class TestController {
	TestView tv = new TestView();
	TestService ts = new TestService();

	public void insertTest(Test t) {
		int result = 0;
		result = ts.insertTest(t);
		
		if(result > 0) {
			System.out.println("방 입력 성공");
		} else {
			tv.displayError("insert");
		}
	}

	public void selectAll() {
		ArrayList<Test> list = ts.selectAll();
		
		if(list.isEmpty()) {
			tv.displayError("select");
		} else {
			tv.printAll(list);
		}
	}

	public void selectName(String n) {
		Test t = ts.selectName(n);
		
		if(t==null) {
			tv.displayError("selectName");
		}else {
			tv.printName(t);
		}
	}

}
```

<br>
<hr>
<br>

## test.model.dao.TestDao

<br>

```java
package test.model.dao;

import java.sql.Statement;
import java.io.FileReader;
import java.io.IOException;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.Properties;

import test.model.vo.Test;
import static common.JDBCTemplate.*;

public class TestDao {

	public int insertTest(Connection con, Test t) {
		int result = 0;
		PreparedStatement pst = null;
		
		try {
			Properties prop = new Properties();
			prop.load(new FileReader("query.properties"));
			String query = prop.getProperty("insertTest");
			
			pst = con.prepareStatement(query);
			pst.setString(1, t.getName());
			pst.setInt(2, t.getNumber());
			pst.setString(3, t.getCode());
			pst.setString(4, t.getEmail());
			result = pst.executeUpdate();
		} catch (IOException e) {
			e.printStackTrace();
		} catch (SQLException e) {
			e.printStackTrace();
		} finally {
			close(pst);
		}
		
		return result;
	}

	public ArrayList<Test> selectAll(Connection con) {
		ArrayList<Test> list = new ArrayList<Test>();
		Statement st = null;
		ResultSet res = null;
		
		
		try {
			Properties prop = new Properties();
			prop.load(new FileReader("query.properties"));
			String query = prop.getProperty("selectAll");
			
			st = con.createStatement();
			res = st.executeQuery(query);
			
			while(res.next()) {
				Test t = new Test();
				t.setName(res.getString(1));
				t.setNumber(res.getInt(2));
				t.setCode(res.getString(3));
				t.setEmail(res.getString(4));
				t.setRoomDate(res.getDate(5));
				
				list.add(t);
			}
			
			
		} catch (IOException e) {
			e.printStackTrace();
		} catch (SQLException e) {
			e.printStackTrace();
		} finally {
			close(res);
			close(st);
		}
		return list;
	}

	public Test selectName(Connection con, String n) {
		PreparedStatement pst = null;
		ResultSet res = null;
		Test t = null;
		
		try {
			Properties prop = new Properties();
			prop.load(new FileReader("query.properties"));
			String query = prop.getProperty("selectName");
			
			pst =con.prepareStatement(query);
			pst.setString(1, n);
			res=pst.executeQuery();
			
			while(res.next()) {
				t = new Test();
				t.setName(res.getString(1));
				t.setNumber(res.getInt(2));
				t.setCode(res.getString(3));
				t.setEmail(res.getString(4));
				t.setRoomDate(res.getDate(5));
			}
		} catch (IOException e) {
			e.printStackTrace();
		} catch (SQLException e) {
			e.printStackTrace();
		} finally {
			close(res);
			close(pst);
		}
		
		return t;
	}

}
```

<br>
<hr>
<br>

## test.service.TestService
<br>

```java
package test.model.service;

import java.sql.Connection;
import java.util.ArrayList;

import test.model.dao.TestDao;
import test.model.vo.Test;
import static common.JDBCTemplate.*;

public class TestService {
	TestDao td = new TestDao();

	public int insertTest(Test t) {
		Connection con = getConnection();
		int result = 0;
		result = td.insertTest(con, t);
		
		if(result>0) {
			commit(con);
		}else {
			rollback(con);
		}
		
		close(con);
		
		return result;
	}

	public ArrayList<Test> selectAll() {
		Connection con = getConnection();
		ArrayList<Test> list = td.selectAll(con);
		
		close(con);
		return list;
	}

	public Test selectName(String n) {
//		Test t = new Test();
		Connection con = getConnection();
		Test t = td.selectName(con,n);
		close(con);
		return t;
	}

}
```

<br>
<hr>
<br>

## test.vo.Test

<br>

```java
package test.model.vo;

import java.sql.Date;

public class Test {
	private String name;
	private int number;
	private String code;
	private String email;
	private Date roomDate;
	
	public Test() {
	}

	public Test(String name, int number, String code, String email, Date roomDate) {
		super();
		this.name = name;
		this.number = number;
		this.code = code;
		this.email = email;
		this.roomDate = roomDate;
	}
	
	

	public Test(String name, int number, String code, String email) {
		super();
		this.name = name;
		this.number = number;
		this.code = code;
		this.email = email;
	}

	public Test(String name) {
		super();
		this.name = name;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public int getNumber() {
		return number;
	}

	public void setNumber(int number) {
		this.number = number;
	}

	public String getCode() {
		return code;
	}

	public void setCode(String code) {
		this.code = code;
	}

	public String getEmail() {
		return email;
	}

	public void setEmail(String email) {
		this.email = email;
	}

	public Date getRoomDate() {
		return roomDate;
	}

	public void setRoomDate(Date roomDate) {
		this.roomDate = roomDate;
	}

	@Override
	public String toString() {
		return "test [name=" + name + ", number=" + number + ", code=" + code + ", email=" + email + ", roomDate="
				+ roomDate + "]";
	}
	
}
```

<br>
<hr>
<br>

## test.view.TestManu

<br>

```java
package test.view;

import java.util.Scanner;

import test.controller.TestController;
import test.model.vo.Test;

public class TestMenu {
	Scanner sc = new Scanner(System.in);

	public void displayMenu() {
		TestController tc = new TestController();

		do {
			System.out.println("===test 이름의 데이터 베이스 관리===");
			System.out.println("1. 새로운 방 추가");
			System.out.println("2. 모든 방 찾기");
			System.out.println("3. 이름으로 방 찾기");
			System.out.println("0. 종료");
			System.out.print("입력 :");
			int num = sc.nextInt();
			sc.nextLine();

			switch (num) {
			case 1:
				tc.insertTest(insertTest());
				break;
			case 2:
				tc.selectAll();
				break;
			case 3:
				tc.selectName(insertTestName());
				break;
			case 0:
				System.out.println("종료합니다");
				return;
			default:
				System.out.println("잘못된 입력입니다.");
			}
		} while (true);
	}

	private Test insertTest() {
		System.out.print("방 이름 :");
		String name = sc.nextLine();
		System.out.print("방 아이디 :");
		int id = sc.nextInt();
		sc.nextLine();
		System.out.print("방 코드 :");
		String code = sc.nextLine();
		System.out.print("이메일 :");
		String email = sc.nextLine();
		return new Test(name, id, code, email);

		// new Test 혹은 Test 객체를 만들고 set으로 내부값을
		// 집어넣어서 만들어준 객체만 리턴하는 방식도 가능

	}

	private String insertTestName() {
		System.out.print("검색할 방 이름 : ");
		String name = sc.nextLine();

		return name;
	}

}
```

<br>
<hr>
<br>

## test.view.TestView

<br>

```java
package test.view;

import java.util.ArrayList;

import test.model.vo.Test;

public class TestView {

	public void displayError(String msg) {
		switch(msg) {
		case "insert" : System.out.println("방 입력 실패!"); break;
		case "select" : System.out.println("전체 출력 실패!"); break;
		default: System.out.println("알 수 없는 오류");
		}
	}

	public void printAll(ArrayList<Test> list) {
		for(Test t : list) {
			System.out.println(t.toString());
		}
	}

	public void printName(Test t) {
		System.out.println(t.toString());
	}

}
```

<br>
<hr>
<br>

## driver.properties

```
driver=oracle.jdbc.driver.OracleDriver
url=jdbc:oracle:thin:@127.0.0.1:1521:xe
user=test
password=test
```

<br>
<hr>
<br>

## query.properties

```
insertTest=insert into test values (?, ?, ?, ?, sysdate)
selectAll=select * from test
selectName=select * from test where name like '%'||?||'%'
```
