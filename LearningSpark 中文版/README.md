### References

1. learning-spark-examples: https://github.com/holdenk/learning-spark-examples

    - [Spark SQL (Java)](https://github.com/holdenk/learning-spark-examples/tree/master/src/main/java/com/oreilly/learningsparkexamples/java)
    - [Mastering Spark SQL](https://jaceklaskowski.gitbooks.io/mastering-spark-sql/content/)
    - [Mastering Apache Spark](https://jaceklaskowski.gitbooks.io/mastering-apache-spark/content/)

2. interactive shell

    - 1.x: `bin/pyspark` or `bin/pyspark --master local[2]`, SparkSession available as 'spark'
    - 2.x: `bin/pyspark` or `bin/pyspark --master local[2]`, SparkContext available as sc, HiveContext available as sqlContext.
    
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
```
### Concepts

1. 每一个spark应用程序 包含 在一个集群上运行各种并行操作的驱动程序(A Spark driver (aka an application's driver process) is a JVM process that hosts SparkContext for a Spark application)

### Spark 1.x vs 2.x

1. [SparkSession vs SparkContext](http://data-flair.training/forums/topic/sparksession-vs-sparkcontext-in-apache-spark)
2. [Apache Spark RDD vs DataFrame vs DataSet](https://data-flair.training/blogs/apache-spark-rdd-vs-dataframe-vs-dataset/)

### Q & A

1. [How to load local file in sc.textFile, instead of HDFS?](https://stackoverflow.com/questions/27299923/how-to-load-local-file-in-sc-textfile-instead-of-hdfs) `lines = sc.textFile("file:///home/XXX/spark-1.6.3-bin-hadoop2.6/README.md")`
