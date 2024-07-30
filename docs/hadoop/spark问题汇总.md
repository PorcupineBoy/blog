



##### Exception in thread "main" org.apache.spark.SparkException: When running with master 'yarn' either HADOOP_CONF_DIR or YARN_CONF_DIR must be set in the environment.

```shell
$ vim /etc/profile
#添加以下两句话
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export YARN_CONF_DIR=$HADOOP_HOME/etc/hadoop
$ :wq
source /etc/profile
```

#####  WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable  Exception in thread "main" java.lang.NoClassDefFoundError: org/apache/spark/sql/DataFrame

```
缺少依赖包
```

#### 查看Spark运行的日志

```
模式 Cluster :需要使用yarn logs 查看日志
模式client 日志会输出到客户端上。
user :dataCenter
applicationId 
yarn logs -applicationId id > logsxx.tex

```

#### Spark: Could not find CoarseGrainedScheduler 异常

```
这个可能是一个资源问题，应该给任务分配更多的 cores 和Executors，并且分配更多的内存。并且需要给RDD分配更多的分区 

找到spark下的conf配置文件中的spark-env.sh

在内容末尾添加配置参数

SPARK_WORKER_CORES=2
SPARK_WORKER_MEMORY=2g
```

#### org.apache.spark.SparkException: A master URL must be set in your configuration

```
未知：
可见这里的匿名内部类依赖类MethodPositionTest$的方法mapFun，所以会触发类MethodPositionTest$的加载以及静态代码块执行，触发报错；

综上，不建议将spark的初始化代码放到main外，很容易出问题。
问题2）当spark相关的初始化代码在main外时，为什么有时报错，有时不报错
具体情形如下：
1）如果main里边的transformation（示例中的map方法）不依赖外部函数调用，正常；
2）如果main里边的transformation（示例中的map方法）依赖main里的函数，报错；
3）如果main里边的transformation（示例中的map方法）依赖main外的函数，报错；
解决：
```

#### java.io.IOException: unexpected exception type

```
WARN TaskSetManager:66 - Lost task 0.0 in stage 0.0 (TID 0, 172.17.190.98, executor 1): java.io.IOException: unexpected exception type
at java.io.ObjectStreamClass.throwMiscException(ObjectStreamClass.java:1736)
at java.io.ObjectStreamClass.invokeReadResolve(ObjectStreamClass.java:1266)
at java.io.ObjectInputStream.readOrdinaryObject(ObjectInputStream.java:2078)
at java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1573)
at java.io.ObjectInputStream.defaultReadFields(ObjectInputStream.java:2287)

解决：
集群上的Scala版本号和打包时工具上的Scala版本号不一致，修改idea上的Scala版本号和集群保持一致。
修改步骤：
1.从File中选择Project Structure
2.选中GlobalLibraries,删除原有Scala，添加需要的Scala版本号，然后选中download
3.重新打包即可


```

all 未能成功加入。

#### 节点丢失问题

```
1、增加reload.sh
```

#### hbase

```
 16:05:39 ERROR executor.Executor: Exception in task 14.1 in stage 0.0 (TID 388)
org.apache.spark.SparkException: Task failed while writing rows
        at org.apache.spark.internal.io.SparkHadoopWriter$.org$apache$spark$internal$io$SparkHadoopWriter$$executeTask(SparkHadoopWriter.scala:155)
        at org.apache.spark.internal.io.SparkHadoopWriter$$anonfun$3.apply(SparkHadoopWriter.scala:83)
        at org.apache.spark.internal.io.SparkHadoopWriter$$anonfun$3.apply(SparkHadoopWriter.scala:78)
        at org.apache.spark.scheduler.ResultTask.runTask(ResultTask.scala:90)
        at org.apache.spark.scheduler.Task.run(Task.scala:121)
        at org.apache.spark.executor.Executor$TaskRunner$$anonfun$11.apply(Executor.scala:407)
        at org.apache.spark.util.Utils$.tryWithSafeFinally(Utils.scala:1408)
        at org.apache.spark.executor.Executor$TaskRunner.run(Executor.scala:413)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
        at java.lang.Thread.run(Thread.java:748)
Caused by: org.apache.hadoop.hbase.client.RetriesExhaustedWithDetailsException: Failed 517 actions: org.apache.hadoop.hbase.RegionTooBusyException: Over memstore limit=512.0M, regionName=6b8012b0496b21b4e54293ac0facb0f0, server=nm-cdh-storage-015.e.189.cn,16020,1584958999992
        at org.apache.hadoop.hbase.regionserver.HRegion.checkResources(HRegion.java:4361)
        at org.apache.hadoop.hbase.regionserver.HRegion.batchMutate(HRegion.java:3980)
        at org.apache.hadoop.hbase.regionserver.HRegion.batchMutate(HRegion.java:3920)
        at org.apache.hadoop.hbase.regionserver.RSRpcServices.doBatchOp(RSRpcServices.java:1034)
        at org.apache.hadoop.hbase.regionserver.RSRpcServices.doNonAtomicBatchOp(RSRpcServices.java:966)
        at org.apache.hadoop.hbase.regionserver.RSRpcServices.doNonAtomicRegionMutation(RSRpcServices.java:929)
        at org.apache.hadoop.hbase.regionserver.RSRpcServices.multi(RSRpcServices.java:2681)
        at org.apache.hadoop.hbase.shaded.protobuf.generated.ClientProtos$ClientService$2.callBlockingMethod(ClientProtos.java:42014)
        at org.apache.hadoop.hbase.ipc.RpcServer.call(RpcServer.java:413)
        at org.apache.hadoop.hbase.ipc.CallRunner.run(CallRunner.java:130)
        at org.apache.hadoop.hbase.ipc.RpcExecutor$Handler.run(RpcExecutor.java:324)
        at org.apache.hadoop.hbase.ipc.RpcExecutor$Handler.run(RpcExecutor.java:304)

```

#### 本地跑spark项目

##### 异常

```
Error: A JNI error has occurred, please check your installation and try again
Exception in thread "main" java.lang.SecurityException: Invalid signature file digest for Manifest main attributes
        at sun.security.util.SignatureFileVerifier.processImpl(Unknown Source)
        at sun.security.util.SignatureFileVerifier.process(Unknown Source)
        at java.util.jar.JarVerifier.processEntry(Unknown Source)
        at java.util.jar.JarVerifier.update(Unknown Source)
        at java.util.jar.JarFile.initializeVerifier(Unknown Source)
        at java.util.jar.JarFile.getInputStream(Unknown Source)
        at sun.misc.JarIndex.getJarIndex(Unknown Source)
        at sun.misc.URLClassPath$JarLoader$1.run(Unknown Source)
        at sun.misc.URLClassPath$JarLoader$1.run(Unknown Source)
        at java.security.AccessController.doPrivileged(Native Method)
        at sun.misc.URLClassPath$JarLoader.ensureOpen(Unknown Source)
        at sun.misc.URLClassPath$JarLoader.<init>(Unknown Source)
        at sun.misc.URLClassPath$3.run(Unknown Source)
        at sun.misc.URLClassPath$3.run(Unknown Source)
        at java.security.AccessController.doPrivileged(Native Method)
        at sun.misc.URLClassPath.getLoader(Unknown Source)
        at sun.misc.URLClassPath.getLoader(Unknown Source)
        at sun.misc.URLClassPath.getNextLoader(Unknown Source)
        at sun.misc.URLClassPath.getResource(Unknown Source)
        at java.net.URLClassLoader$1.run(Unknown Source)
        at java.net.URLClassLoader$1.run(Unknown Source)
        at java.security.AccessController.doPrivileged(Native Method)
        at java.net.URLClassLoader.findClass(Unknown Source)
        at java.lang.ClassLoader.loadClass(Unknown Source)
        at sun.misc.Launcher$AppClassLoader.loadClass(Unknown Source)
        at java.lang.ClassLoader.loadClass(Unknown Source)
        at sun.launcher.LauncherHelper.checkAndLoadMain(Unknown Source)
```

```
解决办法：这是因为在使用Maven打包的时候导致某些包的重复引用，以至于打包之后的META-INF的目录下多出了一些*.SF,*.DSA,*.RSA文件所致，我们可以在pom文件里面加入以下配置：
<<pom.xml>>
  <build>
  <plugins>
  <plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-shade-plugin</artifactId>
    <version>2.2</version>
    <configuration>
      <filters>
        <filter>
          <artifact>*:*</artifact>
          <excludes>
            <exclude>META-INF/*.SF</exclude>
            <exclude>META-INF/*.DSA</exclude>
            <exclude>META-INF/*.RSA</exclude>
          </excludes>
        </filter>
      </filters>
    </configuration>
  </plugin>
  </plugins>
  </build>
```

```
重新打包：
D盘下的tools_0808文件夹为例，打包的文件夹中必须存在MANIFEST.MF文件，存放的位置是

D:\tools_0808\META-INF\MANIFEST.MF。

dos命令如下：

jar cvfm sparkP.jar sparkProject\META-INF\MANIFEST.MF -C sparkProject/ . 
```

```
1、这个是什么需求
2、原定在不同步数据到大数据集群，你的方案、问题、工作量是多少
3、建议把数据同步到集群，工作量、成本是多少

jar cvfm sparkP.jar sparkProject\META-INF\MANIFEST.MF -C sparkProject/ . 

```

#### 本地运行spark，创建视图失败。

```
异常：需要替换sparkmaster的/bin /hadoop.dll  跟wistre.exe 文件
且放置 到系统的System32目录下。
```

#### spark查询hdfs文件是否存在

```
 // sc 为sparkContext;
 val config = sc.hadoopConfiguration
        val fs = org.apache.hadoop.fs.FileSystem.get(config)
        val path = s"/year=${year}/month=${month}/day=${day}"
        val exists = fs.exists(new org.apache.hadoop.fs.Path(ConfigParam.HiveZhczHistory + path))
```

#### shell判断HFDS文件是否存在

```
hadoop fs -test -e /hdfs_dir
if [$? -ne 0];then
	echo "Directory not exists!"
fi

```

```
管理平台
接口调用

每个集群各有两个。接口大概在十几个。
目前：管理平台有一个，新内蒙。
沙溪没有管理平台  ，管理平台在沙溪上也有。

策略， 跟管理平台 要交互， 新增api ，要如果更新 redis.
放内存。 提供一个接口， 写到后台服务器的内存里面。 我可以写到个文件里。
内存的数据。 

权限控制。 参数 层面。  
```

