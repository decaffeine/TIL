#### Spark Test Class (JUnit) 작성/실행    
새 프로젝트 - Spring Legacy Project  

- Guava  
라이브러리 추가  
https://mvnrepository.com/artifact/com.google.guava/guava/24.0-jre  

- src\test\java\WordCountTest.java  
```java
package com.sist.spark;

import org.apache.log4j.Logger;
import org.apache.spark.SparkConf;
import org.apache.spark.api.java.*;
import org.apache.spark.api.java.function.FlatMapFunction;
import org.apache.spark.api.java.function.Function2;
import org.apache.spark.api.java.function.PairFunction;

import java.io.Serializable;
import java.util.*;
import static org.hamcrest.CoreMatchers.is;
import static org.junit.Assert.assertThat;

import org.junit.Before;
import org.junit.After;
import org.junit.Test;

import scala.Tuple2;

public class WordCountTest implements Serializable {

	private static SparkConf conf;
	private static JavaSparkContext sc;
	
	private static Logger log = Logger.getLogger(WordCountTest.class);
	
	@Before
	public void setUp() {
		conf = new SparkConf().setAppName("WordCountTest").setMaster("local[*]");
		sc = new JavaSparkContext(conf);
	}
	
	@After
	public void afterM() {
		if (null != sc) {
			sc.stop();
		}
	}

	@Test
	public void wordTest() {
		List<String> input = new ArrayList<String>();
		input.add("read a book");
		input.add("write a book");
		
		JavaRDD<String> inputRdd = sc.parallelize(input);
		log.info("1 inputRdd = " + inputRdd);
		
		JavaRDD<String> words = inputRdd.flatMap(new FlatMapFunction<String, String>() {

			@Override
			public Iterator<String> call(String t) throws Exception {
				return Arrays.asList(t.split(" ")).iterator();
			}
		});
		log.info("2 words = " + words.collect());

		
		// read a book -> read 1, a 1, book 1
		JavaPairRDD onesWord = words.mapToPair(new PairFunction<String, String, Integer>() {

			@Override
			public Tuple2<String, Integer> call(String t) throws Exception {
				return new Tuple2<String, Integer>(t,1);
			}
		});
		
		log.info("3 onesWord = " + onesWord.keys().collect() + "|" + onesWord.values().collect());

		
		JavaPairRDD<String, Integer> wordCount = onesWord.reduceByKey(new Function2<Integer, Integer, Integer>() {

			@Override
			public Integer call(Integer v1, Integer v2) throws Exception {
				return v1+v2;
			}
		});
		
		log.info("4 wordCount = " + wordCount);
		
		Map<String, Integer> resultMap = wordCount.collectAsMap();

		assertThat(2,is(resultMap.get("book")));
		assertThat(2,is(resultMap.get("a")));
		assertThat(1,is(resultMap.get("read")));
		assertThat(1,is(resultMap.get("write")));
		
		log.info("5 resultMap = " + resultMap.toString());

		
	}//--wordTest

}
```

- WordCountTest 실행 결과 log  
```
INFO : org.apache.spark.SparkContext - Running Spark version 2.1.0
INFO : com.sist.spark.WordCountTest - 1 inputRdd = ParallelCollectionRDD[0] at parallelize at WordCountTest.java:47
INFO : com.sist.spark.WordCountTest - 2 words = [read, a, book, write, a, book]
INFO : com.sist.spark.WordCountTest - 3 onesWord = [read, a, book, write, a, book]|[1, 1, 1, 1, 1, 1]
INFO : com.sist.spark.WordCountTest - 4 wordCount = org.apache.spark.api.java.JavaPairRDD@131fcb6f
INFO : com.sist.spark.WordCountTest - 5 resultMap = {book=2, a=2, write=1, read=1}
```
