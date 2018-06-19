# 180619 _ spark

#### 다운로드  
https://spark.apache.org/downloads.html  

Spark release : 2.1.1 Release  
Pacakge type : Pre-built for Apache Hadoop 2.7 and later  
(설치된 hadoop 버전에 맞추어 선택할 것)  

- checksum 확인 : spark-2.3.0-bin-hadoop2.7.tgz.md5  
```
md5sum spark-2.3.0-bin-hadoop2.7.tgz
```
결과와 같은지 확인  

- 압축 해제, 이동, 링크 만들기  
```
tar xvfz spark-2.3.0-bin-hadoop2.7.tgz
sudo mv spark-2.3.0-bin-hadoop2.7 /usr/local
sudo ln -s spark-2.3.0-bin-hadoop2.7 spark
```

#### 환경설정  
- 환경변수 : /etc/environment  
``` 
path=/usr/local/spark/bin:/usr/local/spark/sbin
SPARK_HOME="/usr/local/spark"
SPARK_SUBMIT="/usr/local/spark/bin/spark-summit"
```

```
source /etc/environment
$PATH
$SPARK_HOME
```

- Spark 설정  

```
cd /usr/local/spark/conf
cp spark-env.sh.template spark-env.sh
cp spark-defaults.conf.template spark-defaults.conf
cp log4j.properties.template log4j.properties
```

  - spark-env.sh
```
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
```

  - spark-defaults.conf  
```
spark.yarn.am.memory=1g
spark.executor.instances=3
```

  - log4j.properties  
```
log4j.rootCategory=WARN, console
```
  
gedit으로 편집했을 때는 찌꺼기를 반드시 지울 것!  


#### Spark 실행

- Hadoop 먼저
```
hadoop namenode -format
start-dfs.sh
start-yarn.sh
jps
```
- 디렉토리 만들고 파일 올리기
```
hdfs dfs -mkdir /input
hdfs dfs -put README.md
```

- PySpark
```
pyspark
```

README.md의 줄수를 세어 보자  
```
>>> lines = sc.textFile("/input/README.md");
>>> lines.count();
```


