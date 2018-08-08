### Docker  
고래가 귀엽다  
컨테이너 기반의 오픈소스 가상화 플랫폼  
  
- 기존의 가상화 방식 : OS를 가상화 (ex. VMware, VirtualBox)  
    - 무겁고 느려서 운영환경에선 사용할 수 없다.  
- 프로세스를 격리하는 방식이 등장  
  
#### Container
- 격리된 공간에서 프로세스가 동작하는 기술  

#### Image  
- Container 실행에 필요한 파일과 설정값 등을 포함  
- 상태값을 가지지 않고 변하지 않는다  
- Container는 Image를 실행한 상태이고 추가하거나 변하는 값은 Container에 저장된다.  

#### Layer  
- Image는 여러 개의 읽기 전용 Layer로 구성  

#### Docker Hub  
- 공짜임 엄청나다

---

#### Docker 설치하고 실행하기  
  
MacOS : Docker for Mac 다운받아 설치, 재부팅 후 실행  
로그인하면 Docker is running… 메시지가 뜬다.  
  
터미널에서
```
docker version
```
  
1. Ubuntu 16.04  
```
docker run ubuntu:16.04
docker run —-rm -it ubuntu:16.04 /bin/bash

cat /etc/environment
ls
```
  
2. tensorflow  
```
docker run -d -p 8888:8888 -p 6006:6006 teamlab/pydata-tensorflow:0.1

localhost:8888/tree
```

```
import tensorflow as tf

hello = tf.constant('Hello, TensorFlow!')
sess = tf.Session()
sess.run(hello)
```

#### 여러가지  Docker 명령어들  

- 컨테이너 목록 확인
```
docker ps
```
  
#### References  
https://subicura.com/2017/01/19/docker-guide-for-beginners-1.html  
https://subicura.com/2017/01/19/docker-guide-for-beginners-2.html
