title: Spark 示例
date: 2016-09-20
tags:
- spark
categories: work
---

#### Spark 处理 Json 数据

##### 需求

1. 源数据在 HDFS 上，数据为非标准 Json 数据，数据样例：

   ``` po
   2016-09-06T10:03:41.913Z 10.69.40.45 {"type":"netflow","stime":"2016-09-01 12:39:45","etime":"2016-09-01 12:49:45","ltime":"2016-09-01 12:50:45","sip":"127.0.0.1","dip":"127.0.0.1","proto":"TCP","sport":445,"dport":80,"ibytes":12345,"obytes":12345}
   ```

2. 读取数据后针对部分字段进行聚合统计次数，产出数据格式为：

   ```po
   IP Time(目前需求是按照小时统计2016-09-07_11:00-12:00) Visit_Count（用户访问量统计）
   ```

3. 产出数据到 HDFS

##### 方案

[Spark 处理 Json 数据](http://www.cnblogs.com/yurunmiao/p/4682315.html)

[Spark Example](http://spark.apache.org/examples.html) -> DataFrame API Examples -> Simple Data Operations & Text Search

```python
from pyspark import SparkContext
from pyspark.sql import SQLContext

if __name__ == "__main__":
        # 创建 Context
        sc = SparkContext(appName="demo")
        # 创建 SQLContext
        sql_context = SQLContext(sc)
        # 读取文本日志，路径为 HDFS 路径
        text_content = sc.textFile("/tmp/test.data")
        # 将文本通过 split 处理，截取 json 字符串后加载为 jsonRDD
        # lambda line: line.split(" ", 2)[2]  通过 lambda 函数将每一行 split，只 split 前 2 个字段
        # [2] 取 split 后的第 3 个字段(从 0 计数)
        json_content = sql_context.jsonRDD(text_content.map(lambda line: line.split(" ", 2)[2]))
        # 以 type & sip 字段进行 groupBy
        result = json_content.groupBy("type", "sip").count()
        # 将结果打印到 Console
        result.show()
        # 将数据存储到 HDFS，路径 HDFS 路径
        result.saveAsTextFile("/tmp/result_test/")
```

若数据为完整的 Json 字符串，实例见下：

```python
from pyspark import SparkContext
from pyspark.sql import SQLContext

if __name__ == "__main__":
        sc = SparkContext(appName="demo")
        sqlContext = SQLContext(sc)
        jsons = sqlContext.jsonFile("/tmp/test.data")
        result = jsons.groupBy("type", "sip").count()
        result.show()
        result.saveAsTextFile("/tmp/result_test/")
```

Spark-submit 提交 python 脚本任务命令：

[Spark 提交 App](http://spark.apache.org/docs/latest/submitting-applications.html)

``` po
# Run application locally
./bin/spark-submit \
  --master local[*] \
  examples/src/main/python/pi.py 1000
# examples/src/main/python/pi.py 为 python 脚本路径
# 1000 为脚本参数
# --master 有 yarn-cluster，yarn-client，local[*]，分别代表着不同的运行模式
```











