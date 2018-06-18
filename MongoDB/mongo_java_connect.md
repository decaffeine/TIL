# 180618  

#### MongoDB - Java 연결
 - 새 프로젝트 : MongoDbEmp  
  Maven Project (quickstart)  

   - 라이브러리 추가 (Java MongoDB Driver)  
https://mvnrepository.com/artifact/org.mongodb/mongo-java-driver/3.6.3

- MongoDbConn.java
```java
package com.sist;

import java.net.InetSocketAddress;
import com.mongodb.BasicDBObject;
import com.mongodb.DB;
import com.mongodb.DBCollection;
import com.mongodb.DBObject;
import com.mongodb.MongoClient;
import com.mongodb.ServerAddress;
/**
 * 
 * $mongo
 * $show dbs
 * $use mydb
 * $show tables
 * $coll = db.getCollection("member");
 * $coll.find()
 * $coll.find({no:1});
 * $coll.update({no:1},{$set:{name:"길동"}});
 * 
 * @author sist
 *
 */
public class MongoDbConn {

	public static void main(String[] args) {
		
		try {
			MongoClient mc = new MongoClient(new ServerAddress(
					new InetSocketAddress("211.238.142.111",27017)
					));
			
			//DB 선택
			DB db = mc.getDB("mydb");
			
			//table 선택
			DBCollection dbc = db.getCollection("member");
			
			//Data Input
			BasicDBObject obj = new BasicDBObject();
			obj.put("no", 8);
			obj.put("name", "다롱이");
			obj.put("user_id", "darong");
			obj.put("passwd", "1234");
			
			dbc.insert(obj);
			System.out.println("==========");
			System.out.println("저장!");
			System.out.println("==========");
			
		}catch(Exception e) {
			System.out.println("=======");
			e.printStackTrace();
			System.out.println("=======");
		}//--try-catch
		

	}//--main

}

```


- MongoDB 터미널에서 먼저 실행한 후 수행
```
mongod -port 27017 -dbpath /usr/local/mongo/data 
```
다른 터미널에서
```
mongo
```

- 데이터 들어갔는지 확인  
```
show dbs;
use mydb
show tables;
coll = db.getCollection("member");
coll.find();
```

- 외부에서 접속 가능하도록 설정  
network interfaces 부분이 외부 접속 관련 설정이다  

/usr/local/mongo/conf/mongodb.conf

```
# mongodb.conf
storage:
  dbPath: /usr/local/mongo/data
  journal:
    enabled: true
#  engine:
#  mmapv1:
#  wiredTiger:

# where to write logging data.
systemLog:
  destination: file
  logAppend: true
  path: /usr/local/mongo/log/mongodb.log

# network interfaces
net:
  port: 27017
  bindIp: 0.0.0.0


#processManagement:

#security:
#        authorization : "enabled"
```

- mongodb 실행시 설정 파일 사용해서 실행
```
mongod --config /usr/local/mongo/conf/mongodb.conf
```

#### csv/json insert

emp.csv -> parsing -> json -> MongoDB

- emp.csv  
```
1001,SIST01,DEV_개발,201806,1000,1020,2F
2001,SIST02,DEV_개발,201806,1000,1020,2F
3001,SIST03,DEV_개발,201806,1000,1020,2F
(생략)
```

- EmpVO.java
```
package com.sist.csv;

public class EmpVO {
/**
 * 1001,SIST01,DEV_개발,201806,1000,1020,2F
2001,SIST02,DEV_개발,201806,1000,1020,2F
 */
	
	private String empno;			//사번
	private String ename;			//이름
	private String job;					
	private String hiredate;		//입사일
	private String sal; 					//급여
	private String deptNo;		//부서번호
	public String getEmpno() {
		return empno;
	}
	public void setEmpno(String empno) {
		this.empno = empno;
	}
	public String getEname() {
		return ename;
	}
	public void setEname(String ename) {
		this.ename = ename;
	}
	public String getJob() {
		return job;
	}
	public void setJob(String job) {
		this.job = job;
	}
	public String getHiredate() {
		return hiredate;
	}
	public void setHiredate(String hiredate) {
		this.hiredate = hiredate;
	}
	public String getSal() {
		return sal;
	}
	public void setSal(String sal) {
		this.sal = sal;
	}
	public String getDeptNo() {
		return deptNo;
	}
	public void setDeptNo(String deptNo) {
		this.deptNo = deptNo;
	}
	@Override
	public String toString() {
		return "EmpVO [empno=" + empno + ", ename=" + ename + ", job=" + job + ", hiredate=" + hiredate + ", sal=" + sal
				+ ", deptNo=" + deptNo + "]";
	}
	
	
	
}

```


- EmpDAO.java
  - insert
```java
package com.sist.csv;

import java.net.InetSocketAddress;
import java.util.ArrayList;
import java.util.List;

import com.mongodb.BasicDBObject;
import com.mongodb.DB;
import com.mongodb.DBCollection;
import com.mongodb.DBCursor;
import com.mongodb.DBObject;
import com.mongodb.MongoClient;
import com.mongodb.ServerAddress;

public class EmpDAO {

	private MongoClient mc;
	private DB db;
	private DBCollection dbc;

	public EmpDAO() {
		try {
			mc = new MongoClient(
					new ServerAddress(new InetSocketAddress("localhost",27017))
					);
			
			db = mc.getDB("mydb");
			dbc = db.getCollection("emp");
			System.out.println("db = "+db);
			System.out.println("dbc = "+dbc);
			
			
		} catch (Exception e) {
			System.out.println("=======");
			e.printStackTrace();
			System.out.println("=======");
		}//--try-catch

	}//--EmpDAO()
	
	/**
	 * table 삭제
	 * @param args
	 */
	public void empDrop() {
		dbc.drop();
	}
	
	/**
	 * insert
	 * csv -> json -> mongo
	 * @param args
	 */
	public void empInsert(EmpVO vo) {
		try {
			BasicDBObject obj=new BasicDBObject();
			/*
			 *  private String empno;//사번
				 private String ename;//이름
				 private String job;//JOb
				 private String hiredate;//입사일
				 private String sal;//급여
				 private String deptNo;//부서번호
			 */
			obj.put("empno", vo.getEmpno());
			obj.put("ename", vo.getEname());
			obj.put("job", vo.getJob());
			obj.put("hiredate", vo.getHiredate());
			obj.put("sal", vo.getSal());
			obj.put("deptNo", vo.getDeptNo());
			
			System.out.println("=========================");
			System.out.println(vo.toString());
			System.out.println("=========================");
			
			this.dbc.insert(obj);
			
		} catch(Exception e) {
			System.out.println("=======");
			e.printStackTrace();
			System.out.println("=======");
		}
		
	}//--empInsert
	
	
	
//	public static void main(String args[]) {
//		EmpDAO empDAO = new EmpDAO();
//	}

}

```

- EmpMain.java  
```java
package com.sist.csv;

import java.io.FileReader;

public class EmpMain {

	public static void main(String[] args) {
		try {
			EmpDAO dao = new EmpDAO();
			dao.empDrop();
			
			//csv file read
			FileReader fr = new FileReader("/home/sist/emp.csv");
			String data = "";
			int i = 0;
			while((i=fr.read()) != -1) {
				data += String.valueOf((char)i);
			}
			
			String[] emp = data.split("\n");
			System.out.println("emp=" + emp.length);
			
			for(i = 0 ; i < emp.length ; i++) {
				String[] value = emp[i].split(",");
				EmpVO vo = new EmpVO();
				vo.setEmpno(value[0]);
				vo.setEname(value[1]);
				vo.setJob(value[2]);
				vo.setHiredate(value[3]);
				vo.setSal(value[4]);
				vo.setDeptNo(value[5]);
				
				dao.empInsert(vo);
			}
		} catch (Exception e) {
			System.out.println("=======");
			e.printStackTrace();
			System.out.println("=======");
		}
	}//--main
}

```

- EmpDAO.java  
  - empAll
```java
/**
	 * 전체 data 조회
	 */
	
	public List<EmpVO> empAll(){
		List<EmpVO> list = new ArrayList<EmpVO>();
		
		try {
			DBCursor cursor = dbc.find();
			/**
			 * BasicDBObject keys = new BasicDBObject();
			 * keys.put("empno","2001");
			 * DBCursor cursor = dbc.find(keys);
			 */
			
			while(cursor.hasNext()) {
				BasicDBObject obj = (BasicDBObject) cursor.next();
				EmpVO vo = new EmpVO();
				
				vo.setEmpno(obj.getString("empno"));
				vo.setEname(obj.getString("ename"));
				vo.setJob(obj.getString("job"));
				vo.setHiredate(obj.getString("hiredate"));
				vo.setSal(obj.getString("sal"));
				vo.setDeptNo(obj.getString("deptNo"));
				
				list.add(vo);

			}
			
			cursor.close();
			
		} catch(Exception e) {
			System.out.println("=======");
			e.printStackTrace();
			System.out.println("=======");
		}
		
		
		return list;
		
	}
```



- MongoJsonMain.java

```java
package com.sist.csv;

import java.util.List;

import com.google.gson.Gson;
import com.google.gson.JsonArray;
import com.google.gson.JsonObject;


public class MongoJsonMain {

	public static void main(String[] args) {

		try {
			EmpDAO dao = new EmpDAO();
			List<EmpVO> list = dao.empAll();
			System.out.println(list);
			
			JsonArray arr = new JsonArray();
			// List -> json  
			for(EmpVO vo : list) {
				JsonObject emp = new JsonObject();
				emp.addProperty("empno", vo.getEmpno());
				emp.addProperty("ename", vo.getEname());
				emp.addProperty("job", vo.getJob());
				emp.addProperty("hiredate", vo.getHiredate());
				emp.addProperty("sal", vo.getSal());
				emp.addProperty("deptNo", vo.getDeptNo());
				
				arr.add(emp);
			}
			
			Gson gson = new Gson();
			String json = gson.toJson(arr);
			System.out.println("***");
			System.out.println("json="+ json);
			
		} catch (Exception e) {
			System.out.println("=======");
			e.printStackTrace();
			System.out.println("=======");
		}//--try-catch

	}//--main

}

```
