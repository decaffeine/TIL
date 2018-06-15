### NoSQL  
- 장단점
  - 많은 서버로 확장(scale-out)이 쉽다, 데이터 유연성  
  - 다양하고 복잡한 데이터 쿼리(join)은 어렵다  

http://lazybrain.ikspres.com/nosql1/

- 종류  
  1.key-value Database (Riak, Voldemort)  
  2.Document Database (mongoDB, CouchDB)  
  3.Big Table Database (Hbase, Casandra)  
  4.Graph Database (Sones,AllegroGraph)  

http://www.devtimes.com/130  

---

### MongoDB  

#### MongoDB vs RDBMS  
|       | RDBMS      | MongoDB            |
|-------|------------|--------------------|
|       | Database   | Database           |
|       | Table      | Collection         |
|       | Tuple/Row  | Document           |
|       | Column     | Key/Value          |
|       | PK         | _id                |
|       | Table Join | Embedded Documents |
| mysql | mysqld     | mongod             |
|       | mysql      | mongo              |

#### 환경설정  
- 다운로드, 압축 풀기  

https://www.mongodb.com/download-center#community  
```
wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-ubuntu1404-3.6.5.tgz  
tar xvfz mongodb-linux-x86_64-ubuntu1404-3.6.5.tgz
```

- 폴더 이동 , 링크 만들기  
```
sudo mv mongodb-linux-x86_64-ubuntu1404-3.6.5 /usr/local/
cd /usr/local
sudo ln -s mongodb-linux-x86_64-ubuntu1404-3.6.5 mongo
```

- path 설정  
```
sudo gedit /etc/environment

PATH=".:/usr/local/mongo/bin:(기존 path)....."

rm /etc/environment~
source /etc/environment
```

- 저장소 만들고 권한 주기  
```
cd /usr/local/mongo
mkdir data
sudo chown sist:sist data
```

- 서비스 실행 (with port번호, dbpath)  
```
mongod -port 27017 -dbpath /usr/local/mongo/data
```
- 다른 터미널 열어서
```
mongo
```

#### 명령어들  

- DB 목록 출력  
```
show dbs
```

```
admin   0.000GB
config  0.000GB
local   0.000GB
```

- DB가 없으면 생성, 있으면 해당 이름의 DB로 선택  
```
use sist
```

- DB 삭제  
```
db.dropDatabase();
```

- collection 생성  
```
db.createCollection("member");
```

- collection 목록
```
show tables
```

- table을 가져왔음  
```
coll = db.getCollection("member");
```

- table의 모든 내용 출력 (select * from ..)
```
coll.find()
```

- insert
```
coll.insert({id:"hyejin01",name:"혜진01",passwd:"1234"});
coll.insert({id:"hyejin02",name:"혜진02",passwd:"1234"});
coll.insert({id:"hyejin03",name:"혜진03",passwd:"1234"});
coll.insert({id:"hyejin04",name:"혜진04",passwd:"1234"});
coll.insert({id:"hyejin05",name:"혜진05",passwd:"1234"});
```

- id가 ~인 것 찾기 (select from where..) 
```
coll.find({id:"hyejin01"});
```

- 삭제  
```
coll.remove({id:"hyejin01"})
```
