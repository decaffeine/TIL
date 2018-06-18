#### 지니뮤직 parsing  
  
지니뮤직에서 제목을 가져와서 youtube에서 뮤직비디오 재생  
http://www.genie.co.kr/chart/top200  
https://www.youtube.com/results?search_query=...  


- 새 프로젝트 : MusicMongoJson
```
Spring Legacy Project  
Templates : Spring MVC Project  
```

- 라이브러리  
  pom.xml에 추가
  - JAXB Core  
https://mvnrepository.com/artifact/com.sun.xml.bind/jaxb-core/2.3.0  
  
  - JSoup  
https://mvnrepository.com/artifact/org.jsoup/jsoup/1.11.2  

- MusicVO.java  
```java
package com.sist.hr;

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

	public static void main(String[] args) {
		MusicManager musicManager = new MusicManager();
		List<MusicVO> list = musicManager.musicAllData();
		System.out.println(list);
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
			
			
			for(int i = 0 ; i < 20 ; i++) {
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

- 결과  
```
[MusicVO [rank=1, title=뚜두뚜두 (DDU-DU DDU-DU), singer=BLACKPINK, poster=//image.genie.co.kr/Y/IMAGE/IMG_ALBUM/081/074/005/81074005_1529044460289_1_140x140.JPG, state=null, id=null, key=IHNzOHi8sJs], 
MusicVO [rank=2, title=Forever Young, singer=BLACKPINK, poster=//image.genie.co.kr/Y/IMAGE/IMG_ALBUM/081/074/005/81074005_1529044460289_1_140x140.JPG, state=null, id=null, key=legSnxdQI6U],
```
