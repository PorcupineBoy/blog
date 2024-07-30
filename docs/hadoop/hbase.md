[Hbase基础知识]



## 启动 

```shell
start-dfs.sh  
start-yarn.sh
zkServer.sh start #每台节点都启动zookeeper
hbase-daemon.sh start master	#master节点启动
hbase-daemon.sh start regionserver  #子节点启动
hbase shell
hbase-daemon.sh stop master #停止
hbase-daemon.sh stop regionserver #停止

```

##  UI

```
默认端口：60010
hdfs:50070
yarn：8088
hbase hbck -fixMeta

1、HDFS页面：50070

2、YARN的管理界面：8088

3、HistoryServer的管理界面：19888

4、Zookeeper的服务端口号：2181

5、Mysql的服务端口号：3306

6、Hive.server1=10000

7、Kafka的服务端口号：9092

8、azkaban界面：8443

9、Hbase界面：16010,60010

10、Spark的界面：8080

11、Spark的URL：7077

```



## 定义

```
hadoop的数据库

hadoop的分布式、开源 、多版本的非关系型数据库

Hbase存储Key-Value格式，面向列存储，Hbase底层为字节数据，没有数据类型一说。
```



## Hbase特性：

- ```
  - 线性和模块化可扩展性
  - 严格一致的读写
  - 表的自动和可配置分片
  - RegionServer之间的自动故障转移支持
  - 方便的基类，用于通过Apache HBase表备份Hadoop MapReduce作业
  - 易于使用的Java API用于客户端访问
  - 块缓存和布隆过滤器用于实时查询
  - 通过服务器端过滤器查询谓词下推
  - Thrift网关和支持XML，Protobuf和二进制数据编码选项的REST-ful Web服务
  - 可扩展的基于Jruby的（JIRB）外壳
  - 支持通过Hadoop指标子系统将指标导出到文件或Ganglia；或通过JMX
  ```

  

## Hbase shell

### hbase 检测

```
1. 检查region是否正常以及修复

#bin/hbase hbck   (检查)
#bin/hbase hbck -fix （修复）
#bin/hbase hbck -fixMeta
#bin/hbase hbck -fixAssignments
#bin/hbase hbck -repair

会返回所有的region是否正常挂载，如没有正常挂载可以使用下一条命令修复，如果还是不能修复，那需要看日志为什么失败，手工处理。

```

```
快照复制之后。新集群的问题：

You need to create a new .regioninfo and region dir in hdfs to plug the hole.

hbase hbck -fixHdfsHoles
hbase hbck -fixAssignments -fixMete

问题：！！！！！！！！！
Exception in thread "main" java.io.IOException: Duplicate hbck - Abort
	at org.apache.hadoop.hbase.util.HBaseFsck.connect(HBaseFsck.java:564)
	at org.apache.hadoop.hbase.util.HBaseFsck.exec(HBaseFsck.java:5119)
	at org.apache.hadoop.hbase.util.HBaseFsck$HBaseFsckTool.run(HBaseFsck.java:4941)
	at org.apache.hadoop.util.ToolRunner.run(ToolRunner.java:70)
	at org.apache.hadoop.util.ToolRunner.run(ToolRunner.java:84)
	at org.apache.hadoop.hbase.util.HBaseFsck.main(HBaseFsck.java:4929)

-encoding UTF-8 -charset UTF-8
```

```
前提：HDFS fsck确保hbase根目录下文件没有损坏丢失，如果有，则先进行corrupt block移除。
切记：一定要在所有Region都上线之后再修复，否则修复之后可能出现重复Region。

步骤1. hbase hbck 检查输出所以ERROR信息，每个ERROR都会说明错误信息。
步骤2. hbase hbck -fixTableOrphans 先修复tableinfo缺失问题，根据内存cache或者hdfs table 目录结构，重新生成tableinfo文件。
步骤3. hbase hbck -fixHdfsOrphans 修复regioninfo缺失问题，根据region目录下的hfile重新生成regioninfo文件。
步骤4. hbase hbck -fixHdfsOverlaps 修复region重叠问题，merge重叠的region为一个region目录，并从新生成一个regioninfo。
步骤5. hbase hbck -fixHdfsHoles 修复region缺失，利用缺失的rowkey范围边界，生成新的region目录以及regioninfo填补这个空洞。
步骤6. hbase hbck -fixMeta 修复meta表信息，利用regioninfo信息，重新生成对应meta row填写到meta表中，并为其填写默认的分配regionserver。
步骤7. hbase hbck -fixAssignments 把这些offline的region触发上线，当region开始重新open 上线的时候，会被重新分配到真实的RegionServer上 , 并更新meta表上对应的行信息。
```



### namespace

```sql
1. list_namespace:查询所有命名空间
hbase(main):001:0> list_namespace
NAMESPACE                                                                       
default                                                                         
hbase

2. list_namespace_tables : 查询指定命名空间的表
hbase(main):014:0> list_namespace_tables 'hbase'
TABLE
meta
namespace

3. create_namespace : 创建指定的命名空间
hbase(main):018:0> create_namespace 'myns'
hbase(main):019:0> list_namespace
NAMESPACE
default
hbase
myns

4. describe_namespace : 查询指定命名空间的结构
hbase(main):021:0> describe_namespace 'myns'
DESCRIPTION
{NAME => 'myns'}


5. alter_namespace ：修改命名空间的结构
hbase(main):022:0>  alter_namespace 'myns', {METHOD => 'set', 'name' => 'eRRRchou'}

hbase(main):023:0> describe_namespace 'myns'
DESCRIPTION
{NAME => 'myns', name => 'eRRRchou'}
修改命名空间的结构=>删除name
hbase(main):022:0> alter_namespace 'myns', {METHOD => 'unset', NAME => 'name'}
hbase(main):023:0> describe_namespace 'myns'

6. 删除命名空间
hbase(main):026:0> drop_namespace 'myns'

hbase(main):027:0> list_namespace
NAMESPACE
default
hbase

7. 利用新添加的命名空间建表
hbase(main):032:0> create 'myns:t1', 'f1', 'f2'

8.查询表快照
hbase(main):033:0> list_snapshot

9.创建快照
hbase(main):033:0>  snapshot 'tablename','snapshotname'
10.删除快照
```

### DDL

```sql
1. 查询所有表
hbase(main):002:0> list
TABLE                                                                           
HelloHbase                                                                      
kylin_metadata                                                                  
myns:t1                                                                         
3 row(s) in 0.0140 seconds

=> ["HelloHbase", "kylin_metadata", "myns:t1"]

2. describe : 查询表结构
hbase(main):003:0> describe 'myns:t1'

{NAME => 'f1', BLOOMFILTER => 'ROW', VERSIONS => '1', IN_MEMORY => 'false', KEEP
_DELETED_CELLS => 'FALSE', DATA_BLOCK_ENCODING => 'NONE', TTL => 'FOREVER', COMP
RESSION => 'NONE', MIN_VERSIONS => '0', BLOCKCACHE => 'true', BLOCKSIZE => '6553
6', REPLICATION_SCOPE => '0'}                                                   
{NAME => 'f2', BLOOMFILTER => 'ROW', VERSIONS => '1', IN_MEMORY => 'false', KEEP
_DELETED_CELLS => 'FALSE', DATA_BLOCK_ENCODING => 'NONE', TTL => 'FOREVER', COMP
RESSION => 'NONE', MIN_VERSIONS => '0', BLOCKCACHE => 'true', BLOCKSIZE => '6553
6', REPLICATION_SCOPE => '0'}

3. 创建分片表
hbase(main):007:0> create 'myns:t2', 'f1', SPLITS => ['10', '20', '30', '40']

4. 修改表，添加修改列簇信息
hbase(main):009:0> alter 'myns:t1', {NAME=>'info1'}
hbase(main):010:0> describe 'myns:t1'

5. 删除列簇
hbase(main):014:0> alter 'myns:t1', {'delete' => 'info1'}
hbase(main):015:0> describe 'myns:t1'

6. 删除表
hbase(main):016:0> disable 'myns:t1'
hbase(main):016:0> enable 'myns:t1'
hbase(main):017:0> drop 'myns:t1'

7.truncate 'zhpt:user_info'
7、负载均衡
hbase(main):017:0> balance

8 创建表:
create 'zhpt:user_info', 'm', SPLITS => ['04444444','08888888','0CCCCCCC','11111110','15555554','19999998','1DDDDDDC','22222220','26666664','2AAAAAA8','2EEEEEEC','33333330','37777774','3BBBBBB8','3FFFFFFC','44444440','48888884','4CCCCCC8','5111110C','55555550','59999994','5DDDDDD8','6222221C','66666660','6AAAAAA4','6EEEEEE8','7333332C','77777770','7BBBBBB4','7FFFFFF8','8444443C','88888880','8CCCCCC4','91111108','9555554C','99999990','9DDDDDD4','A2222218','A666665C','AAAAAAA0','AEEEEEE4','B3333328','B777776C','BBBBBBB0','BFFFFFF4','C4444438','C888887C','CCCCCCC0','D1111104','D5555548','D999998C','DDDDDDD0','E2222214','E6666658','EAAAAA9C','EEEEEEE0','F3333324','F7777768','FBBBBBAC']

```

### DML

```sql
用到的表创建语句：
hbase(main):011:0> create 'myns:user_info','base_info','extra_info'

1. 插入数据（put命令，不能一次性插入多条）
hbase(main):012:0> put 'myns:user_info','001','base_info:username','张三'

2. scan扫描
hbase(main):024:0> scan 'myns:user_info'

3. 通过指定版本查询
hbase(main):024:0> scan 'myns:user_info', {RAW => true, VERSIONS => 1}
hbase(main):024:0> scan 'myns:user_info', {RAW => true, VERSIONS => 2}

4. 查询指定列的数据
hbase(main):014:0> scan 'myns:user_info',{COLUMNS => 'base_info:username'}
4-1. 查询指定列的数据并且过滤
hbase(main):014:0> scan 'myns:user_info',{COLUMNS => 'base_info:username'}，
FILTER =>"ValueFilter(=,'binary:1') "    binary 二进制等于
4-2 前缀过滤查询
 scan 'bill:t_rel_account_base',{ROWPREFIXFILTER=>'000000'}

hbase(main):014:0> scan 'myns:user_info',{COLUMNS => 'base_info:username'}，
FILTER =>"ValueFilter(=,'binary:1')"    binary 二进制等于

5. 分页查询
hbase(main):021:0> scan 'myns:user_info', {COLUMNS => ['base_info:username'], LIMIT => 10, STARTROW => '001'}

6. get查询
hbase(main):015:0> get 'myns:user_info','001','base_info:username'
hbase(main):017:0> put 'myns:user_info','001','base_info:love','basketball'
hbase(main):018:0> get 'myns:user_info','001'

7. 根据时间戳查询 是一个范围，包头不包尾
hbase(main):029:0> get 'myns:user_info','001', {'TIMERANGE' => [1571650017702, 1571650614606]}

8. hbase排序
插入到hbase中去的数据，hbase会自动排序存储：
排序规则：  首先看行键，然后看列族名，然后看列（key）名； 按字典顺序

9. 更新数据
hbase(main):010:0> put 'myns:user_info', '001', 'base_info:name', 'rock'
hbase(main):011:0> put 'myns:user_info', '001', 'base_info:name', 'eRRRchou'

10. incr计数器
hbase(main):053:0> incr 'myns:user_info', '002', 'base_info:age3'

11. 删除
hbase(main):058:0> delete 'myns:user_info', '002', 'base_info:age3'

12. 删除一行
hbase(main):028:0> deleteall 'myns:user_info','001'

13. 删除一个版本
hbase(main):081:0> delete 'myns:user_info','001','extra_info:feature', TIMESTAMP=>1546922931075

14. 删除一个表
hbase(main):082:0> disable 'myns:user_info'
hbase(main):083:0> drop 'myns:user_info'

15. 判断表是否存在
hbase(main):084:0> exists 'myns:user_info'

16. 表生效和失效
hbase(main):085:0> enable 'myns:user_info'
hbase(main):086:0> disable 'myns:user_info'

17. 统计表行数
hbase(main):088:0> count 'myns:user_info'

18. 清空表数据
hbase(main):089:0> truncate 'myns:user_info'
```

## Hbase Java Api

### 依赖

```xml
    <dependencies>
        <dependency>
            <groupId>org.apache.hbase</groupId>
            <artifactId>hbase-client</artifactId>
            <version>1.4.10</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.10</version>
            <scope>compile</scope>
        </dependency>
    </dependencies>
```

### HbaseUtils

```java
public class HbaseUtils {

    public static Configuration configuration = null;
    public static ExecutorService executor = null;
    public static HBaseAdmin hBaseAdmin = null;
    public static Admin admin = null;
    public static Connection conn = null;
    public static Table table;
    static {
        //1. 获取连接配置对象
        configuration = new Configuration();
        //2. 设置连接hbase的参数
        configuration.set("hbase.zookeeper.quorum", "mini01:2181,mini02:2181,mini03:2181");
        //3. 获取Admin对象
        try {
            executor = Executors.newFixedThreadPool(20);
            conn = ConnectionFactory.createConnection(configuration, executor);
            hBaseAdmin = (HBaseAdmin)conn.getAdmin();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static HBaseAdmin getHbaseAdmin(){
        return hBaseAdmin;
    }
    public static Table getTable(TableName tableName) throws IOException {
        return conn.getTable(tableName);
    }
    public static void close(HBaseAdmin admin){
        try {
            admin.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    public static void close(HBaseAdmin admin,Table table){
        try {
            if(admin!=null) {
                admin.close();
            }
            if(table!=null) {
                table.close();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public static void close(Table table){
        try {
            if(table!=null) {
                table.close();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    public static void showResult(Result result) throws IOException {
        CellScanner scanner = result.cellScanner();
        while(scanner.advance()){
            Cell cell = scanner.current();
            System.out.print("\t" + new String(CellUtil.cloneFamily(cell),"utf-8"));
            System.out.print(" : " + new String(CellUtil.cloneQualifier(cell),"utf-8"));
            System.out.print("\t" + new String(CellUtil.cloneValue(cell),"utf-8"));
        }
    }
}
```

### HbaseDemo

```java
public class HbaseDemo {
    private  HBaseAdmin hBaseAdmin = null;
    private  Admin admin = null;
    @Before
    public void init(){
            hBaseAdmin = HbaseUtils.getHbaseAdmin();
    }
    @After
    public void after(){
        HbaseUtils.close(hBaseAdmin);
    }
    @Test
    public void tableExists() throws IOException {  //检查表是否存在
        //4. 检验指定表是否存在，来判断是否连接到hbase
        boolean flag = hBaseAdmin.tableExists("myns:user_info");
        //5. 打印
        System.out.println(flag);
    }

    @Test
    public void listNamespace() throws IOException { //遍历命名空间
        NamespaceDescriptor[] namespaceDescriptors = hBaseAdmin.listNamespaceDescriptors();
        // 打印
        for(NamespaceDescriptor namespaceDescriptor:namespaceDescriptors){
            System.out.println(namespaceDescriptor);
        }
    }

    @Test
    public void listTables() throws Exception{  //获取表的名字
        //获取指定命名空间下的表
        TableName[] tables = hBaseAdmin.listTableNamesByNamespace("myns");
        System.out.println("对应命名空间下的表名：");
        for (TableName table:tables){
            System.out.println(table);
        }
        tables = hBaseAdmin.listTableNames();
        System.out.println("所有表名：");
        for (TableName table:tables){
            System.out.println(table);
        }
    }
    @Test
    public void createNamespace() throws Exception{ //创建namespace
        hBaseAdmin.createNamespace(NamespaceDescriptor.create("eRRRchou").build());
    }

    @Test
    public void createTable() throws Exception{ //创建表
        HTableDescriptor descriptor = new HTableDescriptor(TableName.valueOf("myns:user_info"));
        //创建列簇
        HColumnDescriptor columnDescriptor1 = new HColumnDescriptor("base_info");
        columnDescriptor1.setVersions(1, 5); //设置列簇版本从1到5
        columnDescriptor1.setTimeToLive(24*60*60); //秒
        //创建列簇
        HColumnDescriptor columnDescriptor2 = new HColumnDescriptor("extra_info");
        columnDescriptor2.setVersions(1, 5);
        columnDescriptor2.setTimeToLive(24*60*60); // 秒为单位
        //绑定关系
        descriptor.addFamily(columnDescriptor1);
        descriptor.addFamily(columnDescriptor2);
        //创建表
        hBaseAdmin.createTable(descriptor);
    }
    @Test
    public void deleteTable() throws Exception{ //删除Family
        hBaseAdmin.disableTable("myns:user_info");
        hBaseAdmin.deleteTable("myns:user_info");
    }

    @Test
    public void modifyFamily() throws Exception{ //修改列簇
        TableName tableName = TableName.valueOf("myns:user_info");
        //HTableDescriptor descriptor = new HTableDescriptor(tableName);//原来的列簇消失 new了个新的
        HTableDescriptor descriptor = hBaseAdmin.getTableDescriptor(tableName); //获得原来的描述
        HColumnDescriptor columnDescriptor = new HColumnDescriptor("extra_info");
        columnDescriptor.setVersions(1, 5); //设置列簇版本从1到5
        columnDescriptor.setTimeToLive(24*60*60); //秒
        descriptor.addFamily(columnDescriptor);
        hBaseAdmin.modifyTable(tableName,descriptor);
    }

    @Test
    public void deleteFamily() throws Exception{ //删除Family
        hBaseAdmin.deleteColumn("myns:user_info","extra_info");
    }

    @Test
    public void deleteColumeFamily() throws Exception{ ///删除Family
        TableName tableName = TableName.valueOf("myns:user_info");
        HTableDescriptor tableDescriptor = hBaseAdmin.getTableDescriptor(tableName);
        tableDescriptor.removeFamily("extra_info".getBytes());
        hBaseAdmin.modifyTable(tableName,tableDescriptor);
    }

    @Test
    public void listFamily() throws Exception{  //遍历Family
        TableName tableName = TableName.valueOf("myns:user_info");
        HTableDescriptor tableDescriptor = hBaseAdmin.getTableDescriptor(tableName);
        HColumnDescriptor[] columnFamilies = tableDescriptor.getColumnFamilies();
        for(HColumnDescriptor columnFamilie:columnFamilies){
            System.out.println(columnFamilie.getNameAsString());
            System.out.println(columnFamilie.getBlocksize());
            System.out.println(columnFamilie.getBloomFilterType());
        }
    }

    @Test
    public void getTable() throws IOException {
        Table table = HbaseUtils.getTable(TableName.valueOf("myns:user_info"));
        HbaseUtils.close(table);
    }

    @Test
    public void putDatas() throws IOException {
        Table table = HbaseUtils.getTable(TableName.valueOf("myns:user_info"));
        Put put = new Put(Bytes.toBytes("001"));
        put.addColumn(Bytes.toBytes("base_info"),Bytes.toBytes("userName"),Bytes.toBytes("zhangsan"));
        put.addColumn(Bytes.toBytes("base_info"),Bytes.toBytes("age"),Bytes.toBytes(18));
        put.addColumn(Bytes.toBytes("base_info"),Bytes.toBytes("sex"),Bytes.toBytes("male"));
        //提交
        table.put(put);
        HbaseUtils.close(table);
    }

    @Test
    public void batchPutDatas() throws IOException {
        Table table = HbaseUtils.getTable(TableName.valueOf("myns:user_info"));
        //0. 创建集合
        List<Put> list = new ArrayList<Put>();

        //1. 创建put对象指定行键
        Put rk004 = new Put(Bytes.toBytes("002"));
        Put rk005 = new Put(Bytes.toBytes("003"));
        Put rk006 = new Put(Bytes.toBytes("004"));

        //2. 创建列簇
        rk004.addColumn(Bytes.toBytes("base_info"),Bytes.toBytes("name"),Bytes.toBytes("gaoyuanyuan"));
        rk005.addColumn(Bytes.toBytes("base_info"),Bytes.toBytes("age"),Bytes.toBytes("18"));
        rk005.addColumn(Bytes.toBytes("base_info"),Bytes.toBytes("sex"),Bytes.toBytes("2"));
        rk006.addColumn(Bytes.toBytes("base_info"),Bytes.toBytes("name"),Bytes.toBytes("fanbinbin"));
        rk006.addColumn(Bytes.toBytes("base_info"),Bytes.toBytes("age"),Bytes.toBytes("18"));
        rk006.addColumn(Bytes.toBytes("base_info"),Bytes.toBytes("sex"),Bytes.toBytes("2"));

        //3. 添加数据
        list.add(rk004);
        list.add(rk005);
        list.add(rk006);
        table.put(list);
    }
    @Test
    public void getData() throws Exception{
        Table table = HbaseUtils.getTable(TableName.valueOf("myns:user_info"));
        Get get = new Get(Bytes.toBytes("001"));
        Result result = table.get(get);
        NavigableMap<byte[], byte[]> base_infos = result.getFamilyMap(Bytes.toBytes("base_info"));
        for(Map.Entry<byte[], byte[]> base_info:base_infos.entrySet()){
            String k = new String(base_info.getKey());
            String v = "";
            if(k.equals("age")) {
                 v = String.valueOf(Bytes.toInt(base_info.getValue()));
            }else{
                 v = new String(base_info.getValue());
            }
            System.out.println(k+":"+v);
        }
    }


    @Test
    public void getData2() throws IOException {
        Table table = HbaseUtils.getTable(TableName.valueOf("myns:user_info"));
        //1. 获Get对象
        Get get = new Get(Bytes.toBytes("004"));
        //2. 通过table获取结果对象
        Result result = table.get(get);
        //3. 获取表格扫描器
        CellScanner cellScanner = result.cellScanner();
        System.out.println("rowkey : " + result.getRow());
        //4. 遍历
        while (cellScanner.advance()) {
            //5. 获取当前表格
            Cell cell = cellScanner.current();
            //5.1 获取所有的列簇
            byte[] familyArray = cell.getFamilyArray();
            System.out.println(new String(familyArray, cell.getFamilyOffset(), cell.getFamilyLength()));
            //5.2 获取所有列
            byte[] qualifierArray = cell.getQualifierArray();
            System.out.println(new String(qualifierArray, cell.getQualifierOffset(), cell.getQualifierLength()));
            //5.3 获取所有的值
            byte[] valueArray = cell.getValueArray();
            System.out.println(new String(valueArray, cell.getValueOffset(), cell.getValueLength()));
        }
    }

    @Test
    public void getData3() throws IOException {
        Table table = HbaseUtils.getTable(TableName.valueOf("myns:user_info"));
        //1. 获得Get对象
        Get get = new Get(Bytes.toBytes("004"));
        //2. 通过table获取结果对象
        Result result = table.get(get);
        //3. 获取表格扫描器
        CellScanner cellScanner = result.cellScanner();
        //4.遍历
        while(cellScanner.advance()){
            Cell cell = cellScanner.current();
            //获取所有的列簇
            System.out.println(new String(CellUtil.cloneFamily(cell),"utf8"));
            System.out.println(new String(CellUtil.cloneQualifier(cell),"utf8"));
            System.out.println(new String(CellUtil.cloneValue(cell),"utf8"));
        }
    }

    @Test
    public void batchGetData() throws IOException {
        //1. 创建集合存储get对象
        Table table = HbaseUtils.getTable(TableName.valueOf("myns:user_info"));
        List<Get> gets = new ArrayList<Get>();
        //2. 创建多个get对象
        Get get1 = new Get(Bytes.toBytes("004"));
        get1.addColumn(Bytes.toBytes("base_info"),Bytes.toBytes("name"));
        get1.addColumn(Bytes.toBytes("base_info"),Bytes.toBytes("sex"));
        get1.addColumn(Bytes.toBytes("base_info"),Bytes.toBytes("age"));

        Get get2 = new Get(Bytes.toBytes("001"));
        get2.addColumn(Bytes.toBytes("base_info"),Bytes.toBytes("name"));
        get2.addColumn(Bytes.toBytes("base_info"),Bytes.toBytes("sex"));

        Get get3 = new Get(Bytes.toBytes("003"));
        get3.addColumn(Bytes.toBytes("base_info"),Bytes.toBytes("sex"));
        get3.addColumn(Bytes.toBytes("base_info"),Bytes.toBytes("age"));
        gets.add(get1);
        gets.add(get2);
        gets.add(get3);
        Result[] results = table.get(gets);
        for (Result result:results){
            HbaseUtils.showResult(result);
        }
    }

    @Test
    public void scanTable() throws IOException {
        //1. 创建扫描器
        Scan scan = new Scan();
        //2. 添加扫描的行数包头不包尾
        Table table = HbaseUtils.getTable(TableName.valueOf("myns:user_info"));
        scan.setStartRow(Bytes.toBytes("001"));
        scan.setStopRow(Bytes.toBytes("006" + "\001"));  //小技巧
        //3. 添加扫描的列
        scan.addColumn(Bytes.toBytes("base_info"),Bytes.toBytes("name"));
        //4. 获取扫描器
        ResultScanner scanner = table.getScanner(scan);
        Iterator<Result> it = scanner.iterator();
        while (it.hasNext()){
            Result result = it.next();
            HbaseUtils.showResult(result);
        }
    }
    @Test
    public void deleteData() throws IOException {
        Table table = HbaseUtils.getTable(TableName.valueOf("myns:user_info"));
        //1. 创建集合用于批量删除
        List<Delete> dels = new ArrayList<Delete>();
        //2. 创建删除数据对象
        Delete del = new Delete(Bytes.toBytes("004"));
        del.addColumn(Bytes.toBytes("base_info"),Bytes.toBytes("name"));
        //3. 添加到集合
        dels.add(del);
        //4. 提交
        table.delete(dels);
    }

}
```

### Hbase过滤器

```java
    @Test
    public void filter() throws IOException {
        //RegexStringComparator 正则
        //SubstringComparator; subString比较器
        //BinaryComparator 二进制比较器
        //and条件
        FilterList filterList = new FilterList(FilterList.Operator.MUST_PASS_ALL);
        SingleColumnValueFilter nameFilter = new SingleColumnValueFilter(Bytes.toBytes("base_info"), Bytes.toBytes("name"),
                CompareFilter.CompareOp.LESS_OR_EQUAL,Bytes.toBytes("gaoyuanyuan"));
        filterList.addFilter(nameFilter);
        Scan scan = new Scan();
        scan.setFilter(filterList);
        Table table = HbaseUtils.getTable(TableName.valueOf("myns:user_info"));
        ResultScanner scanner = table.getScanner(scan);
        Iterator<Result> it = scanner.iterator();
        while (it.hasNext()){
            Result result = it.next();
            HbaseUtils.showResult(result);
        }
    }


    @Test
    public void familyFilter() throws IOException {
        //RegexStringComparator 正则
        //SubstringComparator; subString比较器
        //BinaryComparator 二进制比较器
        //and条件
        RegexStringComparator regexStringComparator = new RegexStringComparator("^base");
        //2. 创建FamilyFilter：结果中只包含满足条件的列簇信息
        FamilyFilter familyFilter = new FamilyFilter(CompareFilter.CompareOp.EQUAL, regexStringComparator);



        //4.创建扫描器进行扫描
        Scan scan = new Scan();
        //5. 设置过滤器
        scan.setFilter(familyFilter);
        //6. 获取表对象
        Table table = HbaseUtils.getTable(TableName.valueOf("myns:user_info"));
        //7. 扫描表
        ResultScanner scanner = null;
        try {
            scanner = table.getScanner(scan);
            //8. 打印数据
            Iterator<Result> iterator = scanner.iterator();
            while (iterator.hasNext()) {
                Result result = iterator.next();
                HbaseUtils.showResult(result);
            }
        } catch (IOException e) {
        } finally {
            try {
                table.close();
            } catch (IOException e) {
            }
        }
    }

    @Test
    public void rowFiter() throws IOException {
        //1. 创建RowFilter
        BinaryComparator binaryComparator = new BinaryComparator(Bytes.toBytes("002"));
        RowFilter rowFilter = new RowFilter(CompareFilter.CompareOp.EQUAL, binaryComparator);
        //4.创建扫描器进行扫描
        Scan scan = new Scan();
        //5. 设置过滤器
        scan.setFilter(rowFilter);
        //6. 获取表对象
        Table table = HbaseUtils.getTable(TableName.valueOf("myns:user_info"));
        //7. 扫描表
        ResultScanner scanner = null;
        try {
            scanner = table.getScanner(scan);
            //8. 打印数据
            Iterator<Result> iterator = scanner.iterator();
            while (iterator.hasNext()) {
                Result result = iterator.next();
                HbaseUtils.showResult(result);
            }
        } catch (IOException e) {
        } finally {
            try {
                table.close();
            } catch (IOException e) {
            }
        }
    }
```

#### phoenix-pom

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
   <modelVersion>4.0.0</modelVersion>
   <parent>
      <groupId>xxx.xxx.bigdata.xxxx.datacenter</groupId>
      <artifactId>project-pom</artifactId>
      <version>1.0.1-SNAPSHOT</version>
      <relativePath>../project-pom/pom.xml</relativePath>
   </parent>

   <artifactId>member</artifactId>
   <packaging>jar</packaging>

   <name>member</name>
   <description>会员数据模块</description>

   <properties>
      <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
      <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
      <java.version>1.8</java.version>
   </properties>

   <dependencies>
      <dependency>
         <groupId>xxx.xxx.frame3</groupId>
         <artifactId>common-utils</artifactId>
         <version>${youx-frame3-common.version}</version>
      </dependency>

      <dependency>
         <groupId>xxx.xxx.frame3</groupId>
         <artifactId>common-dao</artifactId>
         <version>${youx-frame3-common.version}</version>
      </dependency>

      <dependency>
         <groupId>xxx.xxx.frame3</groupId>
         <artifactId>common-service</artifactId>
         <version>${youx-frame3-common.version}</version>
      </dependency>

      <dependency>
         <groupId>xxx.xxx.frame3</groupId>
         <artifactId>common-web</artifactId>
         <version>${youx-frame3-common.version}</version>
      </dependency>

      <!-- 查询kylin jdbc driver -->
      <!--<dependency>
         <groupId>org.apache.kylin</groupId>
         <artifactId>kylin-jdbc</artifactId>
         <version>${kylin-jdbc.version}</version>
      </dependency>-->

      <!-- 凤凰hbase插件 -->
      <dependency>
         <groupId>org.apache.phoenix</groupId>
         <artifactId>phoenix-core</artifactId>
         <version>${phoenix-core.version}</version>
         <exclusions>
            <exclusion>
               <groupId>org.slf4j</groupId>
               <artifactId>slf4j-log4j12</artifactId>
            </exclusion>
            <exclusion>
               <groupId>log4j</groupId>
               <artifactId>log4j</artifactId>
            </exclusion>
         </exclusions>
      </dependency>

      <!-- 添加热部署配置 -->
      <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-devtools</artifactId>
         <optional>true</optional>
      </dependency>
   </dependencies>

   <!-- 如果不加下面的插件，打的包不能直接通过jar -jar xxx.jar的方式运行 -->
   <build>
      <plugins>
         <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
         </plugin>

         <!-- 打包jar文件时，配置manifest文件，加入lib包的jar依赖 -->
         <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-jar-plugin</artifactId>
            <configuration>
               <classesDirectory>${project.build.directory}/classes</classesDirectory>
               <archive>
                  <manifest>
                     <mainClass>xxx.xxx.bigdata.xxxx.datacenter.member.MemberApplication</mainClass>
                     <!-- 打包时 MANIFEST.MF文件不记录的时间戳版本 -->
                     <useUniqueVersions>false</useUniqueVersions>
                     <addClasspath>true</addClasspath>
                     <classpathPrefix>lib/</classpathPrefix>
                  </manifest>
                  <manifestEntries>
                     <Class-Path></Class-Path>
                  </manifestEntries>
               </archive>
               <excludes>
                  <!-- 指定打包时要排除的文件,支持正则 -->
                  <exclude>bootstrap.properties</exclude>
               </excludes>
            </configuration>
         </plugin>

         <!-- 把依赖的jar包,打成一个lib文件夹 -->
         <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-dependency-plugin</artifactId>
            <executions>
               <execution>
                  <id>copy-dependencies</id>
                  <phase>package</phase>
                  <goals>
                     <goal>copy-dependencies</goal>
                  </goals>
                  <configuration>
                     <type>jar</type>
                     <includeTypes>jar</includeTypes>
                     <outputDirectory>
                        ${project.build.directory}/lib
                     </outputDirectory>
                  </configuration>
               </execution>
            </executions>
         </plugin>
      </plugins>
   </build>

</project>

```

```
命令：
1.disable '空间名:表名'
2.alter '空间名:表名', METHOD => 'table_att','coprocessor'=>'|org.apache.hadoop.hbase.coprocessor.AggregateImplementation||'
3.enable '空间名:表名'
```

