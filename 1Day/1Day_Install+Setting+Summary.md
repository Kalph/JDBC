목차
=============
* [JDBC](#LocalStorage)<br>
* [기본 세팅](#FileAPI) <br>
* [개요](#BootStrap) <br>

<br>

## JDBC

- 데이터베이스에 접속할 수 있도록 하는 자바 API다. 

<br>

### 설치

<br>

* 방법 1.  oracle.com -> Download -> JDBC Driver -> ojdbc6.jar 다운

* 방법 2.  오라클이 이미 설치되어 있다면 해당 오라클이 설치된 경로의 product\11.2.0\server\jdbc\lib로 이동하면 ojdbc6 파일이 존재함.

<br>

이후 방법1,2든 해당 ojdbc6.jar 파일을 자바 설치된 경로의 jre\lib\ext로 붙여넣기를 진행한다.

<br>
<hr>
<br>

## 기본 세팅

<br>

### <Java>

<br>

이클립스를 실행 후 다음과 같이 설정한다.

<br>

프로젝트 생성 -> properties -> java build path

-> library -> add externam jars -> 방금 추가한 ojbdc6 추가

<br>

* preferences -> general -> workspace -> UTF-8

* general -> editors -> text Editors -> spelling -> default(utf-8)

<br>

### <Oracle>

<br>

* sys as sysdba

<br>

```sql
CREATE USER test IDENTIFIED BY test;
GRANT RESOURCE, CONNECT TO test;
```

<br

* test

<br>

```sql
create table test(
    name varchar2(10),
    id number,
    code varchar2(15),
    email varchar2(15),
    room_date date default sysdate
);

insert into test values('1번방',1,'C','kgg1@or.kr',default);
```

<br>
<hr>
<br>

## 개요

<br>

### < Java >

<br>

* Model.vo.test.java

```java
package model.vo;

import java.sql.Date;

public class test {
	private String name;
	private int number;
	private String code;
	private String email;
	private Date roomDate;
	
	public test() {
	}

	public test(String name, int number, String code, String email, Date roomDate) {
		super();
		this.name = name;
		this.number = number;
		this.code = code;
		this.email = email;
		this.roomDate = roomDate;
	}

	public test(int number, String code, String email, Date roomDate) {
		super();
		this.number = number;
		this.code = code;
		this.email = email;
		this.roomDate = roomDate;
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

* Model.dao.testModel.java

<br>

```java
package model.dao;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

import javax.naming.spi.DirStateFactory.Result;

import model.vo.test;

public class testModel {

	public testModel() {
	}

	public void selectAll() {
		Connection con = null;
		Statement st = null;
		ResultSet res = null;

		try {
			Class.forName("oracle.jdbc.driver.OracleDriver");
			con = DriverManager.getConnection("jdbc:oracle:thin:@localhost:1521:xe", "test", "test");
			String query = "select * from test";
			st = con.createStatement();
			res = st.executeQuery(query);

			while (res.next()) {
				System.out.println(res.getString(1) + ", " + res.getInt(2) + ", " + res.getString(3) + ", "
						+ res.getString(4) + ", " + res.getString(5));
			}
		} catch (ClassNotFoundException e) {
			e.printStackTrace();
		} catch (SQLException e) {
			e.printStackTrace();
		} finally {
			try {
				res.close();
				st.close();
				con.close();
			} catch (SQLException e) {
				e.printStackTrace();
			}
		}
	}

	public void selectOne(String name) {
		Connection con = null;
		Statement st = null;
		ResultSet res = null;
		
		try {
			Class.forName("oracle.jdbc.driver.OracleDriver");
			con = DriverManager.getConnection("jdbc:oracle:thin:@127.0.0.1:1521:xe","test","test");
			
			String query = "select * from test where name='" + name + "'";
			
			st = con.createStatement();
			res = st.executeQuery(query);
			
			while (res.next()) {
				System.out.println(res.getString(1) + ", " + res.getInt(2) + ", " + res.getString(3) + ", "
						+ res.getString(4) + ", " + res.getString(5));
			}
		} catch (ClassNotFoundException e) {
			e.printStackTrace();
		} catch (SQLException e) {
			e.printStackTrace();
		} finally {
			try {
				res.close();
				st.close();
				con.close();
			} catch (SQLException e) {
				e.printStackTrace();
			}
		}
	}

	public void insertTest(test ts) {
		Connection con = null;
		PreparedStatement pst = null;
		int result = 0;
		
		try {
			Class.forName("oracle.jdbc.driver.OracleDriver");
			con = DriverManager.getConnection("jdbc:oracle:thin:@127.0.0.1:1521:xe","test","test");
			
			String query = "insert into test values(?, ?, ?, ?, sysdate)";
			
			pst = con.prepareStatement(query);
			pst.setString(1, ts.getName());
			pst.setInt(2, ts.getNumber());
			pst.setString(3, ts.getCode());
			pst.setString(4, ts.getEmail());
			
			result = pst.executeUpdate();
			
			System.out.println(result + "개의 행 삽입되었습니다.");
			
			if(result > 0){
				con.commit();
			}else {
				con.rollback();
			}
		} catch (ClassNotFoundException e) {
			e.printStackTrace();
		} catch (SQLException e) {
			e.printStackTrace();
		} finally {
			try {
				pst.close();
				con.close();
			} catch (SQLException e) {
				e.printStackTrace();
			}
		}
	}

	public void updateTest(String name3, test ts2) {
		Connection con = null;
		PreparedStatement pst = null;
		int result = 0;
		
		try {
			Class.forName("oracle.jdbc.driver.OracleDriver");
			con = DriverManager.getConnection("jdbc:oracle:thin:@127.0.0.1:1521:xe","test","test");
			
			String query = "update test set email=? where name=?";
			
			pst = con.prepareStatement(query);
			
			pst.setString(1, ts2.getEmail());
			pst.setString(2, name3);
			
			result = pst.executeUpdate();
			
			System.out.println(result + "개의 행이 업데이트됨");
			
			if(result > 0) {
				con.commit();
			}else {
				con.rollback();
			}
		} catch (ClassNotFoundException e) {
			e.printStackTrace();
		} catch (SQLException e) {
			e.printStackTrace();
		} finally {
			try {
				con.close();
				pst.close();
			} catch (SQLException e) {
				e.printStackTrace();
			}
		}
	}

	public void deleteTest(String name3) {
		Connection con = null;
		Statement st = null;
		int res = 0;
		
		try {
			Class.forName("oracle.jdbc.driver.OracleDriver");
			con = DriverManager.getConnection("jdbc:oracle:thin:@127.0.0.1:1521:xe","test","test");
			
			String query = "delete from test where name = '" + name3 + "'";
			st = con.createStatement();
			res = st.executeUpdate(query);
			
			System.out.println(res + "개의 행이 삭제됨");
			
			if(res > 0) {
				con.commit();
			}else {
				con.rollback();
			}
			
		} catch (ClassNotFoundException e) {
			e.printStackTrace();
		} catch (SQLException e) {
			e.printStackTrace();
		}
	}

}
```

<br>

* controller.testJDBC.java

```java
package controller;

import java.util.Scanner;

import model.dao.testModel;
import model.vo.test;

public class TestJDBC {
	public static void main(String[] args) {

		testModel tm = new testModel();
		Scanner sc = new Scanner(System.in);
		int num = 0;

		while (num != 6) {
			System.out.println("1. 모두 출력\n2. 선택한 행 출력\n3. 새 행 추가\n4. 입력받은 방 이름의 이메일만 수정\n5. 선택한 행 삭제\n6. 종료");
			System.out.print("입력 : ");
			num = sc.nextInt();
			sc.nextLine();
			
			switch (num) {
			case 1: 
				tm.selectAll();
				continue;
			case 2: 
				System.out.print("방 이름 입력 : ");
				String name = sc.nextLine();
				tm.selectOne(name);
				continue;
			case 3: 
				System.out.print("방 이름 입력 : ");
				String name2 = sc.nextLine();
				System.out.print("Id 번호 입력 : ");
				int id = sc.nextInt();
				sc.nextLine();
				System.out.print("code 입력 : ");
				String code = sc.nextLine();
				System.out.print("이메일 입력 : ");
				String email = sc.nextLine();
				test ts = new test(name2,id,code,email);
				tm.insertTest(ts);
				continue;
			case 4: 
				System.out.print("방 이름 입력 : ");
				String name3 = sc.nextLine();
				System.out.print("수정할 이메일 입력 : ");
				String email2 = sc.nextLine();
				test ts2 = new test(email2);
				tm.updateTest(name3, ts2);
				continue;
			case 5:
				System.out.print("방 이름 입력 : ");
				String name4 = sc.nextLine();
				tm.deleteTest(name4);
				continue;
			}
			
			System.out.println("종료합니다");
		}
	}

}
```

<br>
