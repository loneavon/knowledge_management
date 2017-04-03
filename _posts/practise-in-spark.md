title: Spark 最佳实践
date: 2016-10-22
tags: Spark
categories: Bigdata
---

#### 介绍

1. 本文介绍的 Spark 基于 HDP 2.4 版本部署，Spark 版本为 2.6.0， Hadoop 版本为 2.7.1。
2. 目前 Spark 的部署方式只支持 Yarn 集群模式
3. 集群 JDK 环境为 1.7，暂不支持 1.8
4. Spark 更详细信息请前往官网: [Spark 官网](http://spark.apache.org/)

#### 开发环境

以 [IntelliJ IDEA](https://www.jetbrains.com/idea/) 作为 IDE，使用 [Maven](https://maven.apache.org/) 作为构建工具。此部分介绍如果构建一个简单的应用

##### 构建项目，添加必要依赖

1. 创建项目

   IntelliJ IDEA 创建项目请参考：[IntelliJ IDEA: Create Maven Project](https://www.jetbrains.com/help/idea/2016.2/getting-started-with-maven.html#create_maven_project)  和 [UsefulDeveloperTools](https://cwiki.apache.org/confluence/display/SPARK/Useful+Developer+Tools#UsefulDeveloperTools-IDESetup)

2. 添加依赖

   在相应的 pom.xml 中添加所需要的基础依赖包，对应的 Maven 配置见下：

   ```xml
   <build>
       <plugins>
         <plugin>
           <groupId>org.scala-tools</groupId>
           <artifactId>maven-scala-plugin</artifactId>
           <version>2.15.2</version>
           <executions>
             <execution>
               <goals>
                 <goal>compile</goal>
               </goals>
             </execution>
           </executions>
         </plugin>
         <plugin>
           <artifactId>maven-compiler-plugin</artifactId>
           <version>3.1</version>
           <configuration>
             <source>1.6</source>
             <target>1.6</target>
           </configuration>
         </plugin>
       </plugins>
   </build>
   <dependencies>
     <dependency>
       <groupId>org.scala-lang</groupId>
       <xrefrtifactId>scala-library</artifactId>
       <version>2.10.2</version>
       <scope>provided</scope>
     </dependency>
     <dependency>
       <groupId>org.apache.spark</groupId>
       <xrefrtifactId>spark-core_2.10</artifactId>
       <version>1.6.0</version>
       <scope>provided</scope>
     </dependency>
   </dependencies>
   ```


以上为基础配置，根据应用使用场景不同，需要添加其他依赖，如使用 Spark Streaming 则需要在 dependencies 中添加 streaming 依赖，具体见下：

```xml
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-streaming_2.10</artifactId>
    <version>1.6.0</version>
</dependency>
```

更多使用请移步 [Getting Started with Maven](https://www.jetbrains.com/help/idea/2016.2/getting-started-with-maven.html)

##### 导出项目

#### Spark 基础操作

##### 提交任务

Spark 通过命令行 spark-submit 来提交任务，具体参数可以通过 spark-submit -help 查看

Spark-submit 提交任务示例：

```po
spark-submit \
  --class org.apache.spark.examples.SparkPi \
  --master local[8] \
  /path/to/examples.jar
# /path/to/examples.jar 为 Jar 包绝对路径
# --class 指定实际调用的类
# --master 有 yarn-cluster，yarn-client，local[*]，分别代表着不同的运行模式
```

更多细节请移步：[Spark Submit Application](https://spark.apache.org/docs/1.4.1/submitting-applications.html)

##### 查看任务状态

查看任务状态可以通过 2 种方式：命令名查看，Web UI 查看

1. 命令行查看

   ```po
   yarn application -list  # 列出目前正在 running 状态的 app，可以指定 App 类型和 App 状态进行过滤
   yarn application -status <Application ID> # 查看某个 App ID 的当前状态
   yarn logs -applicationId <Application ID> [OPTIONS] # 查看 App 具体日志
   ```

   更多细节可通过 yarn -help 查看，或移步 [Yarn Commands](https://hadoop.apache.org/docs/r2.4.1/hadoop-yarn/hadoop-yarn-site/YarnCommands.html#User_Commands) 查看

2. Web UI 查看

   a. 通过控制台下方打开 Ambari 页面

   ![06A9B3E5-E63A-4D74-A5FD-C10674C0D09B](/Users/loneavon/Downloads/06A9B3E5-E63A-4D74-A5FD-C10674C0D09B.png)

   b. Ambari 左侧选择 "Yarn" 后，在右侧上方点击 Active Resource Manager 进入 Yarn Web UI![A9B8F29A-0148-45DC-A3DA-710936677A42](/Users/loneavon/Downloads/A9B8F29A-0148-45DC-A3DA-710936677A42.png)

    c. 进入 Yarn Web UI 后可在左侧边框中选择 Applications 或 Nodes，查看对应 App 或 Node 节点信息 ![79DE660F-3B5F-478C-A710-87112AECD20C](/Users/loneavon/Downloads/79DE660F-3B5F-478C-A710-87112AECD20C.png)

#### 应用开发

应用开发从数据流角度看，分为 3 个阶段分别为读取数据、数据处理、数据导出

##### 读取数据（从不同数据源获取数据示例）

###### Kafka

Spark 提供了 Streaming 模块来对接 Kafka 相关的功能。因 Kafka Consumer API 分为 High Level API 和 Low Level API，在 Spark Streaming 中对应的分别为 KafkaUtils.createStream，KafkaUtils.createDirectStream，本文介绍仅介绍 KafkaUtils.createDirectStream

```scala
  1 // Create context with 2 second batch interval
  2 val sparkConf = new SparkConf().setAppName("DirectKafkaWordCount")
  3 val ssc = new StreamingContext(sparkConf, Seconds(2))
  4
  5 // Create direct kafka stream with brokers and topics
  6 val topicsSet = topics.split(",").toSet
  7 val kafkaParams = Map[String, String]("metadata.broker.list" -> brokers)
  8 val messages = KafkaUtils.createDirectStream[String, String, StringDecoder, StringDecoder](
  9                 ssc, kafkaParams, topicsSet)
```

1. 第 2、3 行创建了 StreamingContext，作为 Streaming 程序的一个入口，其细节请参考 [Spark Streaming Programming Guide](http://spark.apache.org/docs/latest/streaming-programming-guide.html)

2. 第 6 行将需要订阅的 topic 列表按照逗号分隔，示例：

   ```scala
   val topics = "test-topic,test-topic1"
   ```

3. 第 7 行以 K-V 形式设置必要的 Kafka 参数，默认需要添加"metadata.broker.list"，即指定查询元信息的 broker server，示例见下，其他的配置请参考: [Kafka Consumer Configs](http://kafka.apache.org/090/documentation.html#oldconsumerconfigs)

   ```scala
   val brokers = "host1:6667,host2:6667" // KMR 集群 Kafka 默认监听端口为 6667，此处可设置全部 Brokers 的一部分即可
   ```

4. 第 8 行创建一个 DirectStream，返回值 messages 即为当前所获取的数据，后续代码可根据需求此部分数据进行处理

以上为官方 WordCount 的代码片段，若想获取完整代码请移步 [Spark Streaming DirectWorkCount Example](https://github.com/apache/spark/blob/master/examples/src/main/scala/org/apache/spark/examples/streaming/DirectKafkaWordCount.scala)

###### 从 Kafka 读取写入 HDFS 或 KS3

###### 从 Mysql 读取写入 HDFS 或 KS3

```scala
// Creates a DataFrame based on a table named "people"
// stored in a MySQL database.
val url =
  "jdbc:mysql://yourIP:yourPort/test?user=yourUsername;password=yourPassword"
val df = sqlContext
  .read
  .format("jdbc")
  .option("url", url)
  .option("dbtable", "people")
  .load()

// Looks the schema of this DataFrame.
df.printSchema()

// Counts people by age
val countsByAge = df.groupBy("age").count()
countsByAge.show()

// Saves countsByAge to S3 in the JSON format.
countsByAge.write.format("json").save("ks3://...")
// Saves countsByAge to HDFS in the JSON format.
countsByAge.write.format("json").save("hdfs://...")
```

##### 数据处理（数据内容格式处理示例）

###### Json 数据

###### 带有分隔符的文本数据

###### 二进制数据

##### 数据导出（数据持久化到不同存储示例）

###### HDFS

###### Mysql

###### Kafka

###### Hbase

###### KS3

更多示例请参考 [Spark Example](http://spark.apache.org/examples.html)