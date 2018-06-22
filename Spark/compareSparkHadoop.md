#### Hadoop 결과와 Spark 결과 비교하기  

- input (1987.csv의 일부)  
```
1987,10,18,7,729,730,847,849,PS,1451,NA,78,79,NA,-2,-1,SAN,SFO,447,NA,NA,0,NA,0,NA,NA,NA,NA,NA
1987,10,19,1,749,730,922,849,PS,1451,NA,93,79,NA,33,19,SAN,SFO,447,NA,NA,0,NA,0,NA,NA,NA,NA,NA
1987,10,21,3,728,730,848,849,PS,1451,NA,80,79,NA,-1,-2,SAN,SFO,447,NA,NA,0,NA,0,NA,NA,NA,NA,NA
1987,10,22,4,728,730,852,849,PS,1451,NA,84,79,NA,3,-2,SAN,SFO,447,NA,NA,0,NA,0,NA,NA,NA,NA,NA
1987,10,23,5,731,730,902,849,PS,1451,NA,91,79,NA,13,1,SAN,SFO,447,NA,NA,0,NA,0,NA,NA,NA,NA,NA
```

- (main) AirlinePerformanceParser
```java
package com.sist.spark;

public class AirlinePerformanceParser {
	private int year;
	private int month;

	private int arriveDelayTime = 0; //도착 지연
	private int departureDelayTime = 0; //출발 지연
	private int distance = 0; //비행거리
	
	//NA 데이터 처리
	private boolean arriveDelayAvailable = true;
	private boolean departureDelayAvailable = true;
	private boolean distanceAvailable = true;
	
	private String uniqueCarrier; //항공사 코드
	
	public AirlinePerformanceParser(String text) {
		try {
			String[] columns = text.split(",");
			
			year = Integer.parseInt(columns[0]);
			month = Integer.parseInt(columns[1]);
			
			uniqueCarrier = columns[8];
			
			if(!columns[15].equals("NA")) {
				departureDelayTime = Integer.parseInt(columns[15]);
			} else {
				departureDelayAvailable = false;
			}

			if(!columns[14].equals("NA")) {
				arriveDelayTime = Integer.parseInt(columns[14]);
			} else {
				arriveDelayAvailable = false;
			}			

			if(!columns[18].equals("NA")) {
				distance = Integer.parseInt(columns[18]);
			} else {
				distanceAvailable = false;
			}		
			
		} catch (Exception e) {
			System.out.println("AirlinePerformanceParser parsing error:" + e.getMessage());
		}//--try-catch
		
	}

	public int getYear() {
		return year;
	}

	public int getMonth() {
		return month;
	}

	public int getArriveDelayTime() {
		return arriveDelayTime;
	}

	public int getDepartureDelayTime() {
		return departureDelayTime;
	}

	public int getDistance() {
		return distance;
	}

	public boolean isArriveDelayAvailable() {
		return arriveDelayAvailable;
	}

	public boolean isDepartureDelayAvailable() {
		return departureDelayAvailable;
	}

	public boolean isDistanceAvailable() {
		return distanceAvailable;
	}

	public String getUniqueCarrier() {
		return uniqueCarrier;
	}
}

```
1. Spark에서 돌려 보기 

- (test) DepartureDelayTest
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

import com.sist.spark.AirlinePerformanceParser;

public class DepartureDelayTest implements Serializable {
	
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
	public void departueDelayTest() {
		List<String> input = new ArrayList<String>();
		input.add("1987,10,18,7,729,730,847,849,PS,1451,NA,78,79,NA,-2,-1,SAN,SFO,447,NA,NA,0,NA,0,NA,NA,NA,NA,NA");
		input.add("1987,10,19,1,749,730,922,849,PS,1451,NA,93,79,NA,33,19,SAN,SFO,447,NA,NA,0,NA,0,NA,NA,NA,NA,NA");
		input.add("1987,10,21,3,728,730,848,849,PS,1451,NA,80,79,NA,-1,-2,SAN,SFO,447,NA,NA,0,NA,0,NA,NA,NA,NA,NA");
		input.add("1987,10,22,4,728,730,852,849,PS,1451,NA,84,79,NA,3,-2,SAN,SFO,447,NA,NA,0,NA,0,NA,NA,NA,NA,NA ");
		input.add("1987,10,23,5,731,730,902,849,PS,1451,NA,91,79,NA,13,1,SAN,SFO,447,NA,NA,0,NA,0,NA,NA,NA,NA,NA ");
		
		JavaRDD<String> lines = sc.parallelize(input);
		log.info("1. lines = " + lines.collect());
		
		JavaRDD<String> words = lines.flatMap(new FlatMapFunction<String, String>() {

			@Override
			public Iterator<String> call(String t) throws Exception {
				AirlinePerformanceParser parser = new AirlinePerformanceParser(t);
				String yyyyMM = "";
				if(parser.getDepartureDelayTime() > 0) {
					yyyyMM = parser.getYear() + "," + parser.getMonth();
				}
				return Arrays.asList(yyyyMM).iterator();
			}
			
		});
		
		JavaPairRDD<String, Integer> oneWordCount = words.mapToPair(new PairFunction<String, String, Integer>() {

			@Override
			public Tuple2<String, Integer> call(String t) throws Exception {
				return new Tuple2(t,1);
			}
			
		});

		log.info("2. oneWordCount = " + oneWordCount.keys().collect() + "|" + oneWordCount.values().collect());
		
		JavaPairRDD<String, Integer> delayCount = oneWordCount.reduceByKey(new Function2<Integer, Integer, Integer>() {

			@Override
			public Integer call(Integer v1, Integer v2) throws Exception {
				return v1+v2;
			}
			
		});
		
		log.info("3. delayCount = "  + delayCount.keys().collect() + "|" + delayCount.values().collect() );
		
		Map<String, Integer> resultMap = delayCount.collectAsMap();
		
		log.info("4. resultMap = " + resultMap.toString());
	
		assertThat(2,is(resultMap.get("1987,10")));
	
	}
	
	

}

```


- 결과 log  
```
2. oneWordCount = [, 1987,10, , , 1987,10]|[1, 1, 1, 1, 1]
3. delayCount = [, 1987,10]|[3, 2]
4. resultMap = {1987,10=2, =3}
```

2. Hadoop에서 돌려 보기  

- (main) DepartureDelay  
DepartueDelayTest와 대부분 동일하지만 hadoop에서 돌리도록 작성  
```java
package com.sist.spark;

import org.apache.commons.cli.BasicParser;
import org.apache.commons.cli.CommandLine;
import org.apache.commons.cli.CommandLineParser;
import org.apache.commons.cli.Options;
import org.apache.commons.cli.ParseException;
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

import scala.Tuple2;

public class DepartureDelay {
	//private static Logger log = Logger.getLogger(DepartureDelay.class);

	public static void main(String[] args) throws ParseException {
		
		String INPUT_PATH = "";
		  String OUTPUT_PATH = "";
		  
		  //Command param
		  Options options =new Options();
		  options.addOption("i", "input", true, "input path(HDFS)");
		  options.addOption("o", "output", true, "output path(HDFS)");
		  
		  CommandLineParser parser=new BasicParser();
		  CommandLine  cmd = parser.parse(options, args);
		  
		  if(cmd.hasOption("i")) {
		   INPUT_PATH = cmd.getOptionValue("i");
		  }else {
		   System.out.println("INPUT_PATH를 확인 하세요.");
		  }
		  
		  
		  if(cmd.hasOption("o")) {
		   OUTPUT_PATH = cmd.getOptionValue("o");
		  }else {
		   System.out.println("OUTPUT_PATH를 확인 하세요.");
		  }  
		
		SparkConf conf = new SparkConf().setAppName("WordCountTest").setMaster("yarn-cluster");
		JavaSparkContext sc = new JavaSparkContext(conf);
		
		JavaRDD<String> lines = sc.textFile(INPUT_PATH);
		//log.info("1. lines = " + lines.collect());
		
		JavaRDD<String> words = lines.flatMap(new FlatMapFunction<String, String>() {

			@Override
			public Iterator<String> call(String t) throws Exception {
				AirlinePerformanceParser parser = new AirlinePerformanceParser(t);
				String yyyyMM = "";
				if(parser.getDepartureDelayTime() > 0) {
					yyyyMM = parser.getYear() + "," + parser.getMonth();
				}
				return Arrays.asList(yyyyMM).iterator();
			}
			
		});
		
		JavaPairRDD<String, Integer> oneWordCount = words.mapToPair(new PairFunction<String, String, Integer>() {

			@Override
			public Tuple2<String, Integer> call(String t) throws Exception {
				return new Tuple2(t,1);
			}
			
		});

		//log.info("2. oneWordCount = " + oneWordCount.keys().collect() + "|" + oneWordCount.values().collect());
		
		JavaPairRDD<String, Integer> delayCount = oneWordCount.reduceByKey(new Function2<Integer, Integer, Integer>() {

			@Override
			public Integer call(Integer v1, Integer v2) throws Exception {
				return v1+v2;
			}
			
		});
		
		//log.info("3. delayCount = "  + delayCount.keys().collect() + "|" + delayCount.values().collect() );
		
		//Map<String, Integer> resultMap = delayCount.collectAsMap();
		//log.info("4. resultMap = " + resultMap.toString());
		
		delayCount.saveAsTextFile(OUTPUT_PATH);
		sc.stop();

	}

}

```

```
spark-submit --class com.sist.spark.DepartureDelay --master yarn-cluster SparkTest.jar -i /input -o /output
```

- 결과 log  
```
(,740181)
(1987,10,175566)
(1987,12,218858)
```

- Spark의 메모리 기준은 무엇일까?  
