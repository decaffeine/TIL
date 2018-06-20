## 180620   

#### 프로젝트 : SparkWordCount 
New - Maven Project  
Maven Quickstart  

- pom.xml  

```xml
<properties>
	<org.apache.spark.version>2.2.0</org.apache.spark.version>
</properties>

<dependencies>
	<!-- https://mvnrepository.com/artifact/org.apache.spark/spark-core -->
	<dependency>
	    <groupId>org.apache.spark</groupId>
	    <artifactId>spark-core_2.11</artifactId>
	    <version>${org.apache.spark.version}</version>
	    <scope>provided</scope>
	</dependency>

	<!-- https://mvnrepository.com/artifact/org.apache.spark/spark-sql -->
	<dependency>
	    <groupId>org.apache.spark</groupId>
	    <artifactId>spark-sql_2.11</artifactId>
	    <version>${org.apache.spark.version}</version>
	    <scope>provided</scope>
	</dependency>

	<!-- https://mvnrepository.com/artifact/org.apache.spark/spark-streaming -->
	<dependency>
	    <groupId>org.apache.spark</groupId>
	    <artifactId>spark-streaming_2.11</artifactId>
	    <version>${org.apache.spark.version}</version>
	    <scope>provided</scope>
	</dependency>

	<!-- https://mvnrepository.com/artifact/org.apache.spark/spark-mllib -->
	<dependency>
	    <groupId>org.apache.spark</groupId>
	    <artifactId>spark-mllib_2.11</artifactId>
	    <version>${org.apache.spark.version}</version>
	    <scope>provided</scope>
	</dependency>
</dependencies>
```

- Spark RDD  
http://bcho.tistory.com/1027  


- WordCount.java
```java
package com.sist;

import org.apache.commons.cli.*;
import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaPairRDD;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;

import org.apache.spark.api.java.function.FlatMapFunction;
import org.apache.spark.api.java.function.Function2;
import org.apache.spark.api.java.function.PairFunction;

import scala.Tuple2;

import java.util.*;

/**
 * $spark-submit --class com.sist.WordCount --master yarn-cluster SparkWordCount.jar -i /input -o /output
 * @author sist
 *
 */

public class WordCount {
	
	public static void main(String[] args) throws ParseException {
		
		String INPUT_PATH = "";
		String OUTPUT_PATH = "";
		
		//path params from command
		Options options = new Options();
		options.addOption("i", "input", true, "input path(HDFS)");
		options.addOption("o", "output", true, "output path(HDFS)");		
		
		CommandLineParser parser = new BasicParser();
		CommandLine cmd = parser.parse(options, args);
		
		if(cmd.hasOption("i")) {
			INPUT_PATH = cmd.getOptionValue("i");
		} else {
			System.out.println("Input Path를 입력하세요.");
		}
		if(cmd.hasOption("o")) {
			OUTPUT_PATH = cmd.getOptionValue("o");
		} else {
			System.out.println("Output Path를 입력하세요.");
		}
		
		//Spark Context
		SparkConf conf = new SparkConf().setAppName("SparkWordCount").setMaster("yarn-cluster");
		JavaSparkContext context = new JavaSparkContext(conf);
		
		// line -> data를 공백 기준으로 자른 뒤 count
		
		// 한 줄
		JavaRDD<String> lines =context.textFile(INPUT_PATH); 

		//공백 기준으로 word 자르기
		JavaRDD<String> words = lines.flatMap(new FlatMapFunction<String, String>() { 

			@Override
			public Iterator<String> call(String t) throws Exception {
				return Arrays.asList(t.split(" ")).iterator();
			}
		});
		
		// 갯수 세기
		JavaPairRDD<String, Integer> oneWordCount = 	words.mapToPair(new PairFunction<String, String, Integer>() {

			@Override
			public Tuple2<String, Integer> call(String t) throws Exception {
				return new Tuple2<String, Integer>(t,1);
			}
		});
		
		// 더하기
		JavaPairRDD<String, Integer> wordCount = oneWordCount.reduceByKey(
				new Function2<Integer, Integer, Integer>(){

					@Override
					public Integer call(Integer v1, Integer v2) throws Exception {
						return v1+v2;
					}
					
				});
				
		
			wordCount.saveAsTextFile(OUTPUT_PATH);
			context.stop();
	
	}

}

```

- 실행 순서  
  
1. Hadoop 먼저 올리기  
```
hadoop namenode -format
start-dfs.sh
start-yarn.sh
```

  - jps로 확인
```
jps
```

2. 파일 hdfs에 올리기
```
cd /usr/local/spark
hdfs dfs -mkdir /input
```

3. SparkWordCount실행

```
spark-submit --class com.sist.WordCount --master yarn-cluster SparkWordCount.jar -i /input -o /output
```
