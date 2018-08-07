이 책을 보면서 정리함 :  
Spring Boot In Action (번역서- 스프링 부트 코딩 공작소)

### 1. 스프링 시작하기	 

- Boilerplate code : 상용구 코드  
- groovy : java에 python, ruby , smalltalk 등의 특징을 더한 프로그래밍 언어의 한 종류  
    - Gradle : groovy를 이용한 빌드 자동화 시스템  
        - 안드로이드 스튜디오의 공식 빌드 시스템  
        - Java, C++, Python 등 지원  
        - maven의 pom.xml이 gradle의 build.gradle이라고 생각하면 됨  
            - 이해하기 쉬워 보임  

- @RestController = @Controller + @ResponseBody  
    - http://highcode.tistory.com/24  

- 핵심  
    - 1) 자동 구성  
    - 2) 스타터 의존성  
        - 의존성을 기능 바탕으로 가져올 수 있다. (예: springframework.boot:spring-boot-statrter-web)  
        - 스타터가 끌어오는 라이브러리 버전은 이미 호환성 검사가 끝남  
    - 3) CLI(명령줄 인터페이스)  
        - import 없이도?  
    - 4) Actuator  
        - 작동 중인 애플리케이션의 내부를 살펴볼 수 있는 기능  
        - Spring Application Context에 구성된 빈, Spring boot의 자동 구성으로 구성된 것, Application에서 구동 중인 thread의 현재 상태, 최근에 처리된 HTTP 요청 정보, ….

- 오해
    - 1) Application Server가 아니다. 
        - Tomcat 같은 Servlet Container를 내장하고 있을 뿐
	- 2) JPA, JMS 같은 엔터프라이즈 자바 명세를 일절 구현하지 않음
		- 무슨 말이지


### 시작 방법 
- 1. Spring boot CLI
    - mac에서 Homebrew로 설치
```
brew tap pivotal/tap
brew install springboot

spring --version
```

-  2. Spring Initializer
    - 2-1. 웹 기반 인터페이스
        - http://start.spring.io
            - switch to full version 옵션으로 사용 가능한 전체 의존성 목록 확인 가능.
    - 2-2. Spring Tool Suite (STS)
        - Eclipse Marketplace에서 플러그인으로 설치해도 되고
        - http://spring.io/tools/sts 에서 다운로드 후 설치해도 된다.
        - New Spring Starter Project에서 생성
            - STS도 Initializr에 접속하여 프로젝트를 구성하는 것임.
    - 2-3. Spring boot CLI에서 Initalizr 사용
        - -d(초기 의존성) : 공백 없음에 주의
        - -x :  생성된 프로젝트 압축 파일을 현재 디렉토리에 풀기
```
spring init

spring init -dweb,jpa,security

spring init -dweb,jpa,security —build gradle -p jar -x
```

	
