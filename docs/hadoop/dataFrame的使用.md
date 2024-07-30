#### DataFrame详解

```
环境：spark 2.4.0
slaca :2.12以上
```

#### 创建DataFrame的几种方式

```
第一种：rdd 转DF
import session.implict._
val df= rdd.toDF(#columnName)

```

##### 第二种

```
 /**
     * 创建一个空的DataFrame，代表用户
     * 有四列，分别代表ID、名字、年龄、生日
     */
    val colNames = Array("id", "name", "age", "birth")
    //为了简单起见，字段类型都为String
    val schema = StructType(colNames.map(fieldName => StructField(fieldName, StringType, true)))
    //主要是利用了spark.sparkContext.emptyRDD
    val emptyDf = spark.createDataFrame(spark.sparkContext.emptyRDD[Row], schema)

/**
**
**
/
val empty= spark.emptyDataFrame.withColumn()
```



#### DataFrame的方法

```
DataFrame 的函数
Action 操作
1、 collect() ,返回值是一个数组，返回dataframe集合所有的行
2、 collectAsList() 返回值是一个Java类型的数组，返回dataframe集合所有的行
3、 count() 返回一个number类型的，返回dataframe集合的行数
4、 describe(cols: String*) 返回一个通过数学计算的类表值(count, mean, stddev, min, and max)，这个可以传多个参数，中间用逗号分隔，如果有字段为空，那么不参与运算，只这对数值类型的字段。例如df.describe("age", "height").show()
5、 first() 返回第一行 ，类型是row类型
6、 head() 返回第一行 ，类型是row类型
7、 head(n:Int)返回n行  ，类型是row 类型
8、 show()返回dataframe集合的值 默认是20行，返回类型是unit
9、 show(n:Int)返回n行，，返回值类型是unit
10、 table(n:Int) 返回n行  ，类型是row 类型

dataframe的基本操作
1、 cache()同步数据的内存
2、 columns 返回一个string类型的数组，返回值是所有列的名字
3、 dtypes返回一个string类型的二维数组，返回值是所有列的名字以及类型
4、 explan()打印执行计划  物理的
5、 explain(n:Boolean) 输入值为 false 或者true ，返回值是unit  默认是false ，如果输入true 将会打印 逻辑的和物理的
6、 isLocal 返回值是Boolean类型，如果允许模式是local返回true 否则返回false
7、 persist(newlevel:StorageLevel) 返回一个dataframe.this.type 输入存储模型类型
8、 printSchema() 打印出字段名称和类型 按照树状结构来打印
9、 registerTempTable(tablename:String) 返回Unit ，将df的对象只放在一张表里面，这个表随着对象的删除而删除了
10、 schema 返回structType 类型，将字段名称和类型按照结构体类型返回
11、 toDF()返回一个新的dataframe类型的
12、 toDF(colnames：String*)将参数中的几个字段返回一个新的dataframe类型的，
13、 unpersist() 返回dataframe.this.type 类型，去除模式中的数据
14、 unpersist(blocking:Boolean)返回dataframe.this.type类型 true 和unpersist是一样的作用false 是去除RDD

集成查询：
1、 agg(expers:column*) 返回dataframe类型 ，同数学计算求值
df.agg(max("age"), avg("salary"))
df.groupBy().agg(max("age"), avg("salary"))
2、 agg(exprs: Map[String, String])  返回dataframe类型 ，同数学计算求值 map类型的
df.agg(Map("age" -> "max", "salary" -> "avg","phone","count"))
df.groupBy().agg(Map("age" -> "max", "salary" -> "avg"))
3、 agg(aggExpr: (String, String), aggExprs: (String, String)*)  返回dataframe类型 ，同数学计算求值
df.agg(Map("age" -> "max", "salary" -> "avg"))
df.groupBy().agg(Map("age" -> "max", "salary" -> "avg"))
4、 apply(colName: String) 返回column类型，捕获输入进去列的对象
5、 as(alias: String) 返回一个新的dataframe类型，就是原来的一个别名
6、 col(colName: String)  返回column类型，捕获输入进去列的对象
7、 cube(col1: String, cols: String*) 返回一个GroupedData类型，根据某些字段来汇总
8、 distinct 去重 返回一个dataframe类型
9、 drop(col: Column) 删除某列 返回dataframe类型
10、 dropDuplicates(colNames: Array[String]) 删除相同的列 返回一个dataframe
11、 except(other: DataFrame) 返回一个dataframe，返回在当前集合存在的在其他集合不存在的
12、 explode[A, B](inputColumn: String, outputColumn: String)(f: (A) ⇒ TraversableOnce[B])(implicit arg0: scala.reflect.api.JavaUniverse.TypeTag[B]) 返回值是dataframe类型，这个 将一个字段进行更多行的拆分
df.explode("name","names") {name :String=> name.split(" ")}.show();
将name字段根据空格来拆分，拆分的字段放在names里面
13、 filter(conditionExpr: String): 刷选部分数据，返回dataframe类型 df.filter("age>10").show();  df.filter(df("age")>10).show();   df.where(df("age")>10).show(); 都可以
14、 groupBy(col1: String, cols: String*) 根据某写字段来汇总返回groupedate类型   df.groupBy("age").agg(Map("age" ->"count")).show();df.groupBy("age").avg().show();都可以
15、 intersect(other: DataFrame) 返回一个dataframe，在2个dataframe都存在的元素
16、 join(right: DataFrame, joinExprs: Column, joinType: String)
一个是关联的dataframe，第二个关联的条件，第三个关联的类型：inner, outer, left_outer, right_outer, leftsemi
df.join(ds,df("name")===ds("name") and  df("age")===ds("age"),"outer").show();
17、 limit(n: Int) 返回dataframe类型  去n 条数据出来
18、 na: DataFrameNaFunctions ，可以调用dataframenafunctions的功能区做过滤 df.na.drop().show(); 删除为空的行
df.na.fill(0)  为null的填充0
19、 orderBy(sortExprs: Column*) 做alise排序
20、 select(cols:string*) dataframe 做字段的刷选 df.select($"colA", $"colB" + 1)
21、 selectExpr(exprs: String*) 做字段的刷选 df.selectExpr("name","name as names","upper(name)","age+1").show();
22、 sort(sortExprs: Column*) 排序 df.sort(df("age").desc).show(); 默认是asc
23、 unionAll(other:Dataframe) 合并 df.unionAll(ds).show();
24、 withColumnRenamed(existingName: String, newName: String) 修改列表 df.withColumnRenamed("name","names").show();
25、 withColumn(colName: String, col: Column) 增加一列 df.withColumn("aa",df("name")).show();
```

```
四、如何设置合理的分区数

1、分区数越多越好吗？

不是的，分区数太多意味着任务数太多，每次调度任务也是很耗时的，所以分区数太多会导致总体耗时增多。

2、分区数太少会有什么影响？

分区数太少的话，会导致一些结点没有分配到任务；另一方面，分区数少则每个分区要处理的数据量就会增大，从而对每个结点的内存要求就会提高；还有分区数不合理，会导致数据倾斜问题。

3、合理的分区数是多少？如何设置？

总核数=executor-cores * num-executor 

一般合理的分区数设置为总核数的2~3倍

```

```
一 repartition(numPartitions, *cols) 重新分区(内存中，用于并行度)，用于分区数变多，设置变小一般也只是action算子时才开始shuffing;而且当参数numPartitions小于当前分区个数时会保持当前分区个数等于失效。
	df.reparition(100,"month")
1
二 partitionBy(*cols) 根据指定列进行分区（主要磁盘存储，影响生成磁盘文件数量，后续再从存储中载入影响并行度），相似的在一个区，并没有参数来指定多少个分区，而且仅用于PairRdd
df.map(lambda x:(x[0],x[1],x[2])).toDF(["month","day","value"]).partitionBy(["month","day"])
1
三 coalesce(numPartitions) 联合分区，用于将分区变少。不能指定按某些列联合分区
df.coalesce(1).rdd.getNumPartitions()
————————————————
版权声明：本文为CSDN博主「百物易用是苏生」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/u010720408/article/details/90229461
```

#### DataFrame 用法与sql的转化理解

```
data.groupBy("dept").avg("age")
=
select dept ,avg(age) from data group by 1
```

#### DataSet的转化

```
val df=spark.read.json("people.json")
case class Person(name:String ,age int)
val ds:DataSet[Person] =df.as[Person]
val fileterDS=ds.filter(p=>p.age >3)
```

