## Chrome Dev Meetup  

### GDG  
구글 기술에 관심있는 개발자들의 모임  
전세계에!  

GDG Korea Slack Invitation:  
http://slack.gdg.kr  

Slack Link:  
http://gdgkr.slack.com  
WebTech 채널을 찾아주세요.  

GDG Korea WebTech (facebook page)    


---

### Progressive Web Apps   

http://techhtml.github.io/2018/blog/2018/pwa

3~4년 전에 처음 나왔다.
한줄요약 : 웹에서 새로운 사용자 경험을 제공하는 방법론  

#### WHY?  
웹이라고 부르는 환경은 매우 다양하다.  
모바일 자체에 대한 사용성을 진지하게 고민해 본 적이 있나?  

웹이 가지고 있는 특성적인 한계 : 항상 http로 리소스를 요청해서 받아와야 함 (온라인!)  
오프라인에 대한 경험은 제공할 수 없음  

PWA를 사용하고 있다(X)  
PWA를 적용했다/PWA를 적용하기 위해 ~와 같은 feature들을 사용하고 있다.(O)  

#### Current Status of PWA   

iOS에서도 된다!  
IE에서는 안된다... (Edge는 된다)  

Service Worker : PWA에서 가장 핵심적인 feature 중 하나.  

#### FIRE 원칙  
- Fast  
  - 현존하는 모바일 페이지들은 안 빠름. 그냥 한국 LTE가 빠르고 기기가 좋아서 빠르다고 느끼는 것임.  
- Integrated  
- Reliable  
  - 접속 환경이 안 좋더라도(오프라인 홤), 앱이 언제나 동작하여 유저의 신뢰를 깨뜨리지 말아야 함.  
- Engaging  

#### Service Worker    
- 브라우저가 백그라운드에서 실행하는 스크립트
- 웹페이지와는 별개로 동작 (즉, DOM에는 직접 접근 불가)
- 얘를 이용해서 가능한 것 : 푸시 알림, 백그라운드 동기, 캐싱... (즉 오프라인 통제 가능)

- 반드시 https여야 함. (로컬 테스트 시에는 http 가능)
- Promise를 주로 사용함.


- window.addEventListener로 하는 것이 성능상 좋음.

#### Web App Manifest  
- 웹과 앱의 차이점
  - 웹은 브라우저를 통해 접근하고, 주소창이 노출됨.  

- iOS는 webapp manifest를 아직 지원하지 않음. (독자적인 규격)  

- manifest로 Frame 제거도 가능
  - 신기하구만
  - minimul-ui는 사용을 권장하지 않음 (브라우저마다 다르기 때문이다.)

#### PWA 사례 (외국)
http://app.starbucks.com
크롬 개발자 도구에서 ServiceWorker를 살펴보자.  
오프라인 상태에서 새로고침해도 괜찮네!  

당장 적용하기에는 준비할 게 많겠지만...
새로 만든다면 Service Worker 정도는!

---

### CSS Schroll Snap  
https://drafts.csswg.org/css-scroll-snap/

Chrome Update중에 제일 따끈따끈한 기능!
(아직 설치 안 됨!!)

요즘 크롬의 분위기는... 
크롬은 더 이상 W3C Recommendation에 관심이 없다.  
크롬에서 구현하고 싶은 기능을 구현함.  
크롬에(만) 있는 기능을 공부하는 것은 (그것이 웹 표준에 없더라도) 무의미하지 않다!
 - 관리자 페이지 만들면 되지  

#### Scroll Container 
https://drafts.csswg.org/css-overflow-3/  
전체적인 Scroll Snap 정책을 정의 

- scroll-snap-type
  - scroll snap axis
    - x,y : 물리적인 화면의 가로, 세로축
    - block, inline : 언어설정에 따라 축 결정
    - both
 - scroll snap strictness
  - mandatory : 스크롤이 없어도 무조건 적용
  - proximity : 스크롤 양에 따라 적절히 적용  
  - none
- scroll-snap-stop : normal / always
- scroll-padding-TRBL
- scroll-margin-TRBL

#### Snap Element  
- scroll-snap-align : start / end / center

#### Example 1

```html
<div class = "gallery">
<img src=...>
</div>
```

```css
.gallery{
	.display:flex;
	overflow-x : scroll;
	scroll-snap-type: x mandatory;
}


.gallery img{
	scroll-snap-align:center;
}
```

웹 개발은 이제 Declarative(선언하는) 언어로 진행중
우리는 선언만 하면 구현은 크롬 개발팀이 다 해준다

#### Example 2

```html
<article>
	<header> header </header>
	<section style=...> Section 1 </section>
	<section style=...> Section 2 </section>
	<section style=...> Section 3 </section>
</article>
```

```css
html, body {height:100%}
body{margin:0; padding:0}
article {
	height:100%;
	oerflow-y: scroll;
	scroll-snap-type: y mandatory;
	scroll-padding-top: 30px;
}
section {
	height: calc (100% - 30px);
	scroll-snap-align: start;
}
header {
	position: fixed;
	width: 90%; height: 30px;
	top: 0; left: 0;
	background: #559955;
}
section:first-of-type {
	margin-top: 30px;
}
```
