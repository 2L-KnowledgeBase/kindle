### References

1. learning-spark-examples: https://github.com/holdenk/learning-spark-examples

    - [Spark SQL (Java)](https://github.com/holdenk/learning-spark-examples/tree/master/src/main/java/com/oreilly/learningsparkexamples/java)
    - [Mastering Spark SQL](https://jaceklaskowski.gitbooks.io/mastering-spark-sql/content/)
    - [Mastering Apache Spark](https://jaceklaskowski.gitbooks.io/mastering-apache-spark/content/)

2. interactive shell

    - 2.x: `bin/pyspark` or `bin/pyspark --master local[2]`, SparkSession available as 'spark'
    - 1.x: `bin/pyspark` or `bin/pyspark --master local[2]`, SparkContext available as sc, HiveContext available as sqlContext.
    
3. `spark submit`

```
./bin/spark-submit --class org.apache.spark.examples.SparkPi \
--master yarn-cluster \
--num-executors 3 \
--driver-memory 4g \
--executor-memory 2g \
--executor-cores 1 \
lib/spark-examples*.jar \
10

# maven build and run
mvn clean && mvn compile && mvn package
spark-submit --master \
--class com.package_xx.WordCount \
./target/xxx.jar \
./README.md ./wordcounts

# CDH running spark application on YARN
# https://www.cloudera.com/documentation/enterprise/5-11-x/topics/cdh_ig_running_spark_on_yarn.html
spark-submit --class org.apache.spark.examples.SparkPi --master yarn --deploy-mode cluster $SPARK_HOME/lib/spark-examples.jar 10

18/08/27 11:26:16 INFO yarn.Client: Application report for application_1528103907147_4900 (state: ACCEPTED)

yarn logs -applicationId application_1528103907147_4900
```

4. create your own standalone application, e.g. [WordCount](https://github.com/holdenk/learning-spark-examples/blob/master/mini-complete-example/src/main/java/com/oreilly/learningsparkexamples/mini/java/WordCount.java)

``` java
import java.util.Arrays;
import java.util.List;
import java.lang.Iterable;

import scala.Tuple2;

import org.apache.commons.lang.StringUtils;

import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaPairRDD;
import org.apache.spark.api.java.JavaSparkContext;
import org.apache.spark.api.java.function.FlatMapFunction;
import org.apache.spark.api.java.function.Function2;
import org.apache.spark.api.java.function.PairFunction;


public class WordCount {
  public static void main(String[] args) throws Exception {
    String inputFile = args[0];
    String outputFile = args[1];
    
    // Create a Java Spark Context.
    SparkConf conf = new SparkConf().setAppName("wordCount");
		JavaSparkContext sc = new JavaSparkContext(conf);
        
    // Load our input data.
    JavaRDD<String> input = sc.textFile(inputFile);
    
    // Spark 传递函数
    // Split up into words.
    JavaRDD<String> words = input.flatMap(
      new FlatMapFunction<String, String>() {
        public Iterable<String> call(String x) {
          return Arrays.asList(x.split(" "));
        }});
        
    // Transform into word and count.
    JavaPairRDD<String, Integer> counts = words.mapToPair(
      new PairFunction<String, String, Integer>(){
        public Tuple2<String, Integer> call(String x){
          return new Tuple2(x, 1);
        }}).reduceByKey(new Function2<Integer, Integer, Integer>(){
            public Integer call(Integer x, Integer y){ return x + y;}});
    // Save the word count back out to a text file, causing evaluation.
    counts.saveAsTextFile(outputFile);
	}
}
```


### Concepts

1. 每一个spark应用程序 包含 在一个集群上运行各种并行操作的驱动程序(A Spark driver (aka an application's driver process) is a JVM process that hosts SparkContext for a Spark application)
2. spark应用程序在集群运行时的步骤:
    a) 用户使用 spark-submit 提交一个应用
    b) spark-submit 启动 driver 程序，并调用其中的 main() 方法
    c) driver 程序联系 *集群管理器(hadoop YARN/Apache Mesos/built-in Standalong)* 请求资源来启动各 executor
    d) *集群管理器* 代表 driver 程序启动各 executor
    e) driver 进程运行整个用户应用，程序中基于 RDD 的变化和动作（有向无环图 DAG) , driver程序以task形式发送到各 executor
    f) task 在 executor 进程中运行来计算和保存结果
    g) 如果 driver 的 main() 方法退出 或者 调用了 SparkContext.stop(), 就会中止 excuter 的运行并释放集群管理器分配的资源
    


### Spark 1.x vs 2.x

1. [SparkSession vs SparkContext](http://data-flair.training/forums/topic/sparksession-vs-sparkcontext-in-apache-spark)
2. [Apache Spark RDD vs DataFrame vs DataSet](https://data-flair.training/blogs/apache-spark-rdd-vs-dataframe-vs-dataset/)

### Q & A

1. [How to load local file in sc.textFile, instead of HDFS?](https://stackoverflow.com/questions/27299923/how-to-load-local-file-in-sc-textfile-instead-of-hdfs) `lines = sc.textFile("file:///home/XXX/spark-1.6.3-bin-hadoop2.6/README.md")`
2. [http://sqlandhadoop.com/how-to-implement-recursive-queries-in-spark/](http://sqlandhadoop.com/how-to-implement-recursive-queries-in-spark/)
