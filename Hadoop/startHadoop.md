#### Hadoop 시작하고 input 올리기    

1. 하둡 시작  
```
hadoop -namenode format
start-dfs.sh
start-yarn.sh
```
2. DataNode, NodeManager, NameNode, SecondaryNameNode, ResourceManager  
모두 올라와 있는지 확인  
```
jps 
```

3. input 디렉토리 만들고 1987.csv~2008.csv 모두 /input에 올리기  
```
hdfs dfs -mkdir /input
cd dataexpo
hdfs dfs -put *.csv /input
```
