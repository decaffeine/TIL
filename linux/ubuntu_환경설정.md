# 180521  
  
#### 배경  
  
우분투 14.04에서 노트북 무선 랜 카드(qca61x4)를 지원하지 않아서 16.04로 새로 설치  
와이파이도 설치 과정에서 바로 잡히고 사운드도 바로 나온다!!

---

#### 우분투 16.04 설치하기 (윈도우 듀얼 부팅)  
  
http://doocong.com/webserver/ubuntu-setup/  
파티션은 따로 건드리지 않고 윈도우와 함께 설치를 선택

----

#### 한글 입력 & 한영키 설정  
  
http://androidtest.tistory.com/52
http://interp.blog/우분투-fcitx-한영키-한자키-매핑

14.04에서는 iBus를 사용했으나 16.04부터는 fcitx를 사용하는 것 같다  
한영키 매핑이 안 될 때는(나는 오른쪽 alt로 읽힘) 두 번째 포스팅의 'gnome-tweak-tool' 사용

---

#### 크롬 설치  
  
https://brunch.co.kr/@hancoma/90  
그냥 설치하면 설치가 안 된다  
여기 보고 따라하기


#### jdk 설치  
  
http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html   

1. 리눅스용 jdk 파일 다운로드 후 압축 풀기  
2. jdk 폴더를 가능하면 한글이 포함되지 않은 폴더 (나의 경우 /usr/local) 로 옮겨 주
3. sudo gedit /etc/environment  
  
```
JAVA_HOME="/usr/local/jdk1.8.0_171"
path = ".:/usr/local/jdk1.8.0_171/bin:(원래있던내용....)"
```

4. 
```
cd /etc
ls -lart
```
만약 environment~ 파일이 있다면(찌꺼기) 
```
sudo rm environment~
```

5. 변경한 내용 적용  
```
source /etc/environment
```

6. 확인  
```
echo $JAVA_HOME
java -version
```
  

#### STS 설치   
https://spring.io/tools   
가서 다운받고 압축 풀면 된다

#### STS 설치 후 환경설정  
1. Windows - Preferences - keys 검색 - content 검색 -   
content Assist - Binding을 Ctrl + space로 바꾸고 저장  
2. Tomcat 설치  
https://github.com/decaffeine/TIL/blob/master/Spring/180517.md#새-프로젝트--spring_12  

  
#### 카카오톡 설치  
  
1. wine 설치  
https://wiki.winehq.org/Ubuntu  
나는 stable 버전으로 설치했다 

2. playonlinux 설치  
소프트웨어 센터에서 설치함  
(apt transaction returned result exit-failed 설치 실패가 나올 때는  
https://extrememanual.net/12670  
시스템 설정 - 소프트웨어 & 업데이트 - 다운로드 위치 : 주 서버로 변경)  

3. playonlinux 환경에서 카카오톡 설치   
https://ncube.net/13771


#### Numix 테마 설치  
https://yeopbox.com/우분투-메이트-16-04-lts-numix-테마-및-나눔고딕-글꼴-적용하기  


