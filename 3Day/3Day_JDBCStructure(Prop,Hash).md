<br>

## 이전에 사용했던 내용 계속해서 이어 감.

<br>

## test.view 

<br>

```java
	public void displayName(HashMap<String, Test> hash) {
		Iterator<String> it = hash.keySet().iterator();
		if (!hash.isEmpty()) {
			while (it.hasNext()) {
				System.out.println(hash.get(it.next()));
			}
		}else {
			System.out.println("해당 이름이 없습니다.");
		}

	}
```

<hr>
<br>
<hr>

## test.controller.TestController

<br>

```java
	public void selectName(String n) {
		HashMap<String, Test> hash = ts.selectName(n);
		
		tv.displayName(hash);
	}
```

<hr>
<br>
<hr>

## test.model.service.TestService

<br>

```java
	public HashMap<String, Test> selectName(String n) {
		Connection con = getConnection();
		HashMap<String, Test> hash = td.selectName(con, n);
		
		close(con);
		return hash;
	}
```

<br>
<hr>
<br>

## test.model.dao.TestDao

```java
public HashMap<String, Test> selectName(Connection con, String n) {
		HashMap<String, Test> hash = new HashMap<String, Test>();
		PreparedStatement pst = null;
		ResultSet res = null;
		
		Properties prop = new Properties();
		try {
			prop.load(new FileReader("query.properties"));
			String query = prop.getProperty("selectName");
			
			pst = con.prepareStatement(query);
			pst.setString(1, n);
			res = pst.executeQuery();
			
			while(res.next()) {
				Test t = new Test();
				t.setName(res.getString(1));
				t.setNumber(res.getInt(2));
				t.setCode(res.getString(3));
				t.setEmail(res.getString(4));
				t.setRoomDate(res.getDate(5));
				
				hash.put(t.getName(), t);
			}
		} catch (IOException e) {
			e.printStackTrace();
		} catch (SQLException e) {
			e.printStackTrace();
		}
		return hash;
	}
```

<br>
