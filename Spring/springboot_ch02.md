### 2. 첫 번째 스프링 부트 애플리케이션 개발하기

- H2DB?
    - 자바  기반 오픈소스 RDBMS
    - 별도의 설치 과정이 없고 용량도 2MB (압축 버전 기준)
    - JDBC API 지원, 표준 SQL 대부분 지원
    - 개발 단계에서 test DB로 많이 사용됨

- 프로젝트 환경 설정
    - Dependencies
        - Web, Thymeleaf, JPA, H2
    - Spring CLI
```
spring init -d=web,thymeleaf,data-jpa,h2 --groupId=com.manning --artifactId=readingList --name="Reading List" --package-name=readingList --description="Reading List Demo" --build gradle readingList
```

---

### 갓 초기화한 스프링 부트 프로젝트
cf. bootstrap은 사전적으로 ‘예비 명령에 의해 프로그램을 로드하는 방법’이란 뜻이 있다.

#### @SpringBootApplication

- ReadingListApplication.java
```java
package readingList;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class ReadingListApplication {

    public static void main(String[] args) {
        SpringApplication.run(ReadingListApplication.class, args);
    }
}


```

- @SpringBootApplication : 스프링 컴포넌트 검색, 스프링 부트 자동 구성 활성화
아래 3가지 annotation을 묶은 것임
	- 스프링의 @Configuration
		- 이것이 붙은 클래스를 스프링의 자바 기반 구성 클래스로 지정
	- 스프링의 @ComponentScan
		- 컴포넌트 검색 기능을 활성화하여 웹 컨트롤러나 다른 컴포넌트 클래스들을 자동 검색, Spring context에 bean으로 등록
	- 스프링 부트의 @EnableAutoConfiguration
		- 스프링 부트의 자동 구성 기능

#### Test

- ReadingListApplicationTests.java (책)

```java
package readingList;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.boot.test.SpringApplicationConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.test.context.web.WebbAppConfiguration;

@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiugration(classes=ReadingListApplication.class)
@WebAppConfiguration
public class ReadingListApplicationTests {

    @Test
    public void contextLoads() {
    }

}
```

- (Spring Initializr로 만든 파일)
```java
package readingList;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest
public class ReadingListApplicationTests {

    @Test
    public void contextLoads() {
    }

}

```
- 원래 스프링에서는 @ContextConfiguration 을 썼다.  
하지만 스프링 부트에서는 @SpringApplicationConfiguration을 사용해야 함(?)  
- contextLoads만으로도 애플리케이션 콘텍스트가 잘 로드되었는지 확인 가능.

? 책에 쓰여 있는 코드와 Spring Initializr로 나온 테스트 코드의 내용이 다름

#### application.properties
지금은 빈 파일
```
server.port=8000
```
이라고 쓰면 내장 톰캣 서버가 8000번 포트를 이용함.  
(application.properties를 로드하라고 스프링 부트에 명시적으로 요청한 적이 없는데도 그렇게 함)  

### 빌드
빌드를 제대로 해 본 적이 없었다니!  
예전에 했던 스프링 웹 프로젝트(maven) 들을 빌드해 보자  




### References
- [H2DB](http://dololak.tistory.com/285)
	
