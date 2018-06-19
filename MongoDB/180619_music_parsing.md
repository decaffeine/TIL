# 180619 

- DAO에서는 단건만  

- MusicDAO.java  
```java
package com.sist.hr;

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

public class MusicDAO {
	private MongoClient mc;
	private DB db;
	private DBCollection dbc;
	
	public MusicDAO() {
		
		try {
		mc = new MongoClient(new ServerAddress(
						new InetSocketAddress("localhost",27017)
				));
		
		db = mc.getDB("mydb");
		dbc = db.getCollection("music");
		
		} catch (Exception e) {
			System.out.println("====");
			e.printStackTrace();
			System.out.println("====");
			
		}
	}
	
	/**
	 * 테이블 삭제
	 */
	public void musicDrop() {
		dbc.drop();
	}
	
	
	
	public void do_save(MusicVO vo) {
		try {
			
			System.out.println("vo=" + vo.toString());

			BasicDBObject obj = new BasicDBObject();
			obj.put("rank", vo.getRank());
			obj.put("title", vo.getTitle());
			obj.put("singer", vo.getSinger());
			obj.put("poster", vo.getPoster());
			obj.put("state", vo.getState());
			obj.put("id",vo.getId());
			obj.put("key", vo.getKey());
			
			dbc.insert(obj);
			
		} catch (Exception e) {
			System.out.println("====");
			e.printStackTrace();
			System.out.println("====");

		}
	}
	
	public List<MusicVO> do_selectList() {
		List<MusicVO> list = new ArrayList<MusicVO>();

		try {
			
			//==검색 조건 넣고 싶을 때==
			//BasicDBObject keys = new BasicDBObject();
			//keys.put("empno","2001");
			//dbc.find(keys); 

			DBCursor cursor = dbc.find();
			
			while(cursor.hasNext()) {
				BasicDBObject obj = (BasicDBObject) cursor.next();
				MusicVO vo = new MusicVO();
				
				vo.setId(obj.getString("id"));
				vo.setKey(obj.getString("key"));
				vo.setPoster(obj.getString("poster"));
				vo.setRank(obj.getInt("rank"));
				vo.setSinger(obj.getString("singer"));
				vo.setState(obj.getString("state"));
				vo.setTitle(obj.getString("title"));
				
				list.add(vo);
			}
			
			cursor.close();
			
		} catch (Exception e) {
			System.out.println("====");
			e.printStackTrace();
			System.out.println("====");
		}
		return list;
	}

}
```

- MusicManager.java  
```java
package com.sist.hr;

import java.util.ArrayList;
import java.util.List;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

import org.jsoup.Jsoup;
import org.jsoup.nodes.Document;
import org.jsoup.select.Elements;

public class MusicManager {
	
	public List<MusicVO> do_selectList() {
		List<MusicVO> list = new ArrayList<MusicVO>();
		try {

			MusicDAO dao = new MusicDAO();
			list = dao.do_selectList();
			
		} catch (Exception e) {
			System.out.println("====");
			e.printStackTrace();;
			System.out.println("====");
			
		}
		return list;
	}
	
	public int do_save(List<MusicVO> list) {
		try {
			MusicDAO dao = new MusicDAO();
			dao.musicDrop();
			
			for(MusicVO vo : list) {
				dao.do_save(vo);
			}
		
		} catch (Exception e) {
			System.out.println("====");
			e.printStackTrace();
			System.out.println("====");
		}
		return 0;
	}

	public static void main(String[] args) {
		MusicManager musicManager = new MusicManager();
//		List<MusicVO> list = musicManager.musicAllData();
		

//		int saveFlag = musicManager.do_save(list);
//		System.out.println("saveFlag=" + saveFlag);
		
		List<MusicVO> listAll = musicManager.do_selectList();
		System.out.println("listAll="+ listAll);
		
		
		
	}//--main
	
	public String youtube(String title) {
		
		//{"url":"/watch?v=IHNzOHi8sJs"
		
		String key = "";
		String url = "https://www.youtube.com/results?search_query=";
		try {
			Document doc = Jsoup.connect(url+title).get();
			Pattern p = Pattern.compile("/watch\\?v=[^가-힣]+");
			Matcher m = p.matcher(doc.toString());
			while(m.find()) {
				String data = m.group();
				//System.out.println("data=" + data);
				//data=/watch?v=9II_GQJ7mKo"
				key = data.substring(data.indexOf("=")+1,data.indexOf("\""));
				break;
			}
			
		} catch (Exception e) {
			System.out.println("=====");
			e.printStackTrace();
			System.out.println("=====");
		}//--try-catch

		
		return key;
	}
	
	/**
	 * genieMusic parsing
	 * @return
	 */
	public List<MusicVO> musicAllData(){
		List<MusicVO> list = new ArrayList<MusicVO>();
		
		try { 
			Document doc = Jsoup.connect("http://www.genie.co.kr/chart/top200").get();
			
			//<a href="#" class="cover" onclick="fnViewAlbumLayer('81074005');return false;">
			//<span class="mask"></span><img src="//image.genie.co.kr/Y/IMAGE/IMG_ALBUM/081/074/005/81074005_1529044460289_1_140x140.JPG" 
			Elements poster = doc.select("a.cover img");

			//<a href="#" class="title ellipsis" title="재생" onclick="fnPlaySong('88040624','1');return false;">
			Elements title = doc.select("table.list-wrap a.title");
					
			Elements singer = doc.select("table.list-wrap a.artist");
			
//			Elements rank = doc.select("table.list-wrap tbody td.number");
			
			
			for(int i = 0 ; i < 10 ; i++) {
//				System.out.println( rank.get(i).text()
//											+ "|| \t" + singer.get(i).text() 
//											+ "\t" + title.get(i).text() 
//											+ "\t" + poster.get(i).attr("src"));
				
				MusicVO vo = new MusicVO();
				vo.setRank(i+1);
				vo.setSinger(singer.get(i).text());
				vo.setPoster(poster.get(i).attr("src"));				
				vo.setTitle(title.get(i).text());

				vo.setKey(youtube(vo.getTitle()));

				list.add(vo);
			}
			
			
			
		} catch (Exception e) {
			System.out.println("=====");
			e.printStackTrace();
			System.out.println("=====");
		}//--try-catch
		
		return list;
	}

}

```

- home 디렉토리의 .bash_history 파일에  
터미널에서 수행했던 모든 명령어가 남아 있다.

```
mongod --config /usr/local/mongo/conf/mongodb.conf
```

```
$ mongo
> use mydb
> show tables
> coll = db.getCollection("music");
> coll.find();
> coll.find().pretty();
```

```
{ "_id" : ObjectId("5b284eb12dcca616165c7104"), "rank" : 1, "title" : "뚜두뚜두 (DDU-DU DDU-DU)", "singer" : "BLACKPINK", "poster" : "//image.genie.co.kr/Y/IMAGE/IMG_ALBUM/081/074/005/81074005_1529044460289_1_140x140.JPG", "state" : null, "id" : null, "key" : "IHNzOHi8sJs" }

{
	"_id" : ObjectId("5b284eb12dcca616165c7104"),
	"rank" : 1,
	"title" : "뚜두뚜두 (DDU-DU DDU-DU)",
	"singer" : "BLACKPINK",
	"poster" : "//image.genie.co.kr/Y/IMAGE/IMG_ALBUM/081/074/005/81074005_1529044460289_1_140x140.JPG",
	"state" : null,
	"id" : null,
	"key" : "IHNzOHi8sJs"
}

```
---

#### 새 프로젝트 : MusicMongo  
Spring Legacy Project  
Template : Spring MVC Project  

| controller                | service                             | dao                |
|---------------------------|-------------------------------------|--------------------|
|                           | MusicService                        |                    |
| MusicController           | MusicServiceImple                   | MusicDAO           |
| com.sist.music.controller | com.sist.music.service              | com.sist.music.dao |
| do_selectList()           | do_selectList()                     | do_save(MusicVO)   |
| do_saveAll()              | do_saveAll(){    do_save(MusicVO) } | do_selectList()    |

- servlet-context.xml  
```xml
	<!-- Resolves views selected for rendering by @Controllers to .jsp resources in the /WEB-INF/views directory -->
	<beans:bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<beans:property name="prefix" value="/" />
		<beans:property name="suffix" value=".jsp" />
	</beans:bean>
```

- web.xml  

```xml
	<servlet-mapping>
		<servlet-name>appServlet</servlet-name>
		<url-pattern>*.do</url-pattern>
	</servlet-mapping>

 <!-- 한글 변환 코드 -->
  <filter>
    <filter-name>encodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
       <param-name>encoding</param-name>
       <param-value>UTF-8</param-value>
    </init-param>
  </filter>
  <filter-mapping>
    <filter-name>encodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
```

- MusicDao.java  
```java
package com.sist.music.dao;

import java.net.InetSocketAddress;
import java.util.ArrayList;
import java.util.List;


import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Repository;

import com.mongodb.BasicDBObject;
import com.mongodb.DB;
import com.mongodb.DBCollection;
import com.mongodb.DBCursor;
import com.mongodb.MongoClient;
import com.mongodb.ServerAddress;

import com.sist.music.domain.MusicVO;


@Repository
public class MusicDao {
	
	Logger log = LoggerFactory.getLogger(this.getClass());
	
	private MongoClient mc;
	private DB db;
	private DBCollection dbc;
	
	public MusicDao() {
		
		try {
		mc = new MongoClient(new ServerAddress(
						new InetSocketAddress("localhost",27017)
				));
		
		db = mc.getDB("mydb");
		dbc = db.getCollection("music");
		
		} catch (Exception e) {
			log.debug("====");
			e.printStackTrace();
			log.debug("====");
			
		}
	}
	
	/**
	 * 테이블 삭제
	 */
	public void musicDrop() {
		dbc.drop();
	}
	
	
	
	public void do_save(MusicVO vo) {
		try {
			
			log.debug("vo=" + vo.toString());

			BasicDBObject obj = new BasicDBObject();
			obj.put("rank", vo.getRank());
			obj.put("title", vo.getTitle());
			obj.put("singer", vo.getSinger());
			obj.put("poster", vo.getPoster());
			obj.put("state", vo.getState());
			obj.put("id",vo.getId());
			obj.put("key", vo.getKey());
			
			dbc.insert(obj);
			
		} catch (Exception e) {
			log.debug("====");
			e.printStackTrace();
			log.debug("====");

		}
	}
	
	public List<MusicVO> do_selectList() {
		List<MusicVO> list = new ArrayList<MusicVO>();

		try {
			
			//==검색 조건 넣고 싶을 때==
			//BasicDBObject keys = new BasicDBObject();
			//keys.put("empno","2001");
			//dbc.find(keys); 

			DBCursor cursor = dbc.find();
			
			while(cursor.hasNext()) {
				BasicDBObject obj = (BasicDBObject) cursor.next();
				MusicVO vo = new MusicVO();
				
				vo.setId(obj.getString("id"));
				vo.setKey(obj.getString("key"));
				vo.setPoster(obj.getString("poster"));
				vo.setRank(obj.getInt("rank"));
				vo.setSinger(obj.getString("singer"));
				vo.setState(obj.getString("state"));
				vo.setTitle(obj.getString("title"));
				
				list.add(vo);
			}
			
			cursor.close();
			
		} catch (Exception e) {
			log.debug("====");
			e.printStackTrace();
			log.debug("====");
		}
		return list;
	}
}

```

- MusicVO.java  
```java
package com.sist.music.domain;

public class MusicVO {
	private int			rank ;  //순위
	private String 	title;  //제목
	private String 	singer;  //가수
	private String 	poster;  //사진
	private String 	state;  //상태
	private String 	id;  //id
	private String 	key;  //key
	
	
	public int getRank() {
		return rank;
	}
	public void setRank(int rank) {
		this.rank = rank;
	}
	public String getTitle() {
		return title;
	}
	public void setTitle(String title) {
		this.title = title;
	}
	public String getSinger() {
		return singer;
	}
	public void setSinger(String singer) {
		this.singer = singer;
	}
	public String getPoster() {
		return poster;
	}
	public void setPoster(String poster) {
		this.poster = poster;
	}
	public String getState() {
		return state;
	}
	public void setState(String state) {
		this.state = state;
	}
	public String getId() {
		return id;
	}
	public void setId(String id) {
		this.id = id;
	}
	public String getKey() {
		return key;
	}
	public void setKey(String key) {
		this.key = key;
	}
	@Override
	public String toString() {
		return "MusicVO [rank=" + rank + ", title=" + title + ", singer=" + singer + ", poster=" + poster + ", state="
				+ state + ", id=" + id + ", key=" + key + "]";
	}

	

}

```

- MusicService.java  
```java
package com.sist.music.service;

import java.util.List;

import com.sist.music.domain.MusicVO;

public interface MusicService {

	public void do_saveAll();
	
	public List<MusicVO> do_selectList() ;
}
```

- MusicServiceImple.java  
```java
package com.sist.music.service;

import java.util.ArrayList;
import java.util.List;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

import org.jsoup.Jsoup;
import org.jsoup.nodes.Document;
import org.jsoup.select.Elements;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import com.sist.music.dao.MusicDao;
import com.sist.music.domain.MusicVO;

@Service
public class MusicServiceImple implements MusicService {

	Logger log = LoggerFactory.getLogger(this.getClass());
	
	@Autowired
	private MusicDao dao;

	@Override
	public void do_saveAll() {
		List<MusicVO> list = musicAllData();
		
		try {

			dao.musicDrop();
			
			for(MusicVO vo : list) {
				dao.do_save(vo);
			}
		
		} catch (Exception e) {
			log.debug("====");
			e.printStackTrace();
			log.debug("====");
		} //--try-catch
		
	}


	@Override
	public List<MusicVO> do_selectList() {
		return dao.do_selectList();
	}


	/**
	 * genieMusic parsing
	 * @return
	 */
	public List<MusicVO> musicAllData(){
		List<MusicVO> list = new ArrayList<MusicVO>();
		
		try { 
			Document doc = Jsoup.connect("http://www.genie.co.kr/chart/top200").get();
			
			Elements poster = doc.select("a.cover img");
			Elements title = doc.select("table.list-wrap a.title");
			Elements singer = doc.select("table.list-wrap a.artist");
			
			for(int i = 0 ; i < 10 ; i++) {

				MusicVO vo = new MusicVO();
				vo.setRank(i+1);
				vo.setSinger(singer.get(i).text());
				vo.setPoster(poster.get(i).attr("src"));				
				vo.setTitle(title.get(i).text());

				vo.setKey(youtube(vo.getTitle()));

				list.add(vo);
			}
			
		} catch (Exception e) {
			log.debug("=====");
			e.printStackTrace();
			log.debug("=====");
		}//--try-catch
		
		return list;
	}
	
	private String youtube(String title) {

		String key = "";
		String url = "https://www.youtube.com/results?search_query=";
		try {
			Document doc = Jsoup.connect(url + title).get();
			Pattern p = Pattern.compile("/watch\\?v=[^가-힣]+");
			Matcher m = p.matcher(doc.toString());
			while (m.find()) {
				String data = m.group();
				key = data.substring(data.indexOf("=") + 1, data.indexOf("\""));
				break;
			}

		} catch (Exception e) {
			log.debug("=====");
			e.printStackTrace();
			log.debug("=====");
		} // --try-catch

		return key;
	}
	
}

```

- MusicController.java

```java
package com.sist.music.controller;

import java.util.List;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.ResponseBody;

import com.google.gson.*;
import com.sist.music.domain.MusicVO;
import com.sist.music.service.MusicService;

@Controller
public class MusicController {
	
	Logger log = LoggerFactory.getLogger(this.getClass());
	
	@Autowired
	private MusicService musicService;
	
	@RequestMapping(value = "do_selectList.do", method = RequestMethod.GET,
			 produces="application/json;charset=UTF-8")
	@ResponseBody
	public String do_selectList() {
		List<MusicVO> list = musicService.do_selectList();
		
		// List -> Json
		JsonArray arr = new JsonArray();
		for(MusicVO vo : list) {
			JsonObject music = new JsonObject();
			music.addProperty("rank", vo.getRank());
			music.addProperty("title", vo.getTitle());
			music.addProperty("state",vo.getState());
			music.addProperty("singer", vo.getSinger());
			music.addProperty("poster", vo.getPoster());
			music.addProperty("key", vo.getKey());
			music.addProperty("id", vo.getId());
			
			arr.add(music);
		}
		
		Gson gson = new Gson();
		String json = gson.toJson(arr);
		log.debug("***" + json);
		return json;
		
	}

}
```
