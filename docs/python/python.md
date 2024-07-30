#### 图片转base64

```
def img_to_BASE64(path):
    with open(path, 'rb') as f:
        base64_data = base64.b64encode(f.read())
        return base64_data
```

```
情况说明：
图表添加至邮件中：
有两种方式实现:
1、在一个web服务器中添加相应代码，将数据渲染成图表，而后添加到邮件中，发送邮件。
2、利用python3当中的pyecharts模块进行图表开发，且保存成图片而后添加至邮件中，发送邮件。

问题1、：没有web服务器可用。(该服务器必须为云数据的机器)，不能因为图表添加专门开发一个项目吧。
问题2、：集群上没有python3 的环境，也没有pyecharts相应开发所需要导入的开发环境和模块，我无权限。
```

```
安装完python3之后
需要安装phantomjs
$- yum install npm


安装---发行版的python3 ==anaconda 是一个用于科学计算的 Python 发行版
https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/Anaconda3-2020.02-Linux-x86_64.sh

$source activate  进入 python3 模式   
$source deaactivate  退出模式。
---安装phantomjs

$ pip install pyecharts
$pip install snapshot_phantomjs
$下载 phantomjs_压缩包  解压
$tar -xjvf phantomjs-linux.tar.br2 
$mv phantomjs-linxu /opt/phantomjs
$vim /etc/profile  #添加路径
 path=/opt/phantomjs/bin
 --保存退出
 source /etc/profile #刷新环境变量
 ---测试
 $phtomjs -v  
 成功
```

#### Mapplot

#### Pycharts

```
保存图片为png 之后图片过大。
保存图片为其他格式时，背景图为黑色。解决办法：pip install pillow 
通过在背景上替换白色背景。之后另存为新图片。
############
图表处理过程：
1、利用pyecharts生成httml ，之后利用phantomjs 生成png文件。
2、重置png图表大小，之后利用PIL(pillow)模块对图片进行白色背景处理。
3、将数据转为base64。
```

#### DataFrame

```
连接mysql 获取到的数据 转化 dataFrame
import pandas as pd
##pd.DataFrame(list(results))

获取某一列数据 
#df['columnName']  
#df2.loc[:,'two']  # 取出标签为'two'的列数据
#df2.iloc[:,1]   # 取出第2列数据
loc函数与iloc函数的区别：

loc是通过标签来获取数据，loc[‘a’]取出标签为’a’的行，loc[:,‘two],取出标签为‘two’的列数据，loc[‘a’,‘two’]取出行标签为’a’，列标签为’two’的数据
iloc是通过位置来获取数据，iloc[1]取出第二行，iloc[:,1]取出第2列数据，iloc[1,1] 指取出第2行第2列的数据

获取行数据 =>通过切片方式获取
df[1:3] //第二行跟第三行
```

#### 字符串拼接的两种方式

```
sql = "SELECT channel,day,mau_total,lmau,mau_increase_percent FROM t_zhpt_kpi_daily_info \
       WHERE DAY between %s and %s " % (dateBegin, dateEnd)

sql = "SELECT channel,day,mau_total,lmau,mau_increase_percent FROM t_zhpt_kpi_daily_info \
       WHERE DAY between {} and {} ".format(dateBegin, dateEnd)
```

#### python数据类型转化

```
在python中：

字符串str转换成int: int_value = int(str_value)

int转换成字符串str: str_value = str(int_value)

int -> unicode: unicode(int_value)

unicode -> int: int(unicode_value)

str -> unicode: unicode(str_value)

unicode -> str: str(unicode_value)

int -> str: str(int_value)

str -> int: int(str_value)
```

```
字体微软雅黑。 图80% 。 增加总的
```

#### python数据库连接

```
# -*- coding: utf-8 -*-
import pymysql
import pandas as pd
#显示所有列
pd.set_option('display.max_columns', None)
config={
	'host':'127.0.0.1',
	'port':3306,
	'passwd':'135790',
	'charset':'utf8mb4',
	'cursorclass': pymysql.cursors.DictCursor
}
conn=pymysql.connect(**config)
conn.autocommit(1)
conn.select_db("mydb")
cursor=conn.cursor()
sql="select *from table"
try:
	results=cursor.curson.fetchall()
	print(results)
	ds=pd.DataFrame(list(results))
catch{
	case ex:FileNotFoundException->{
	print("Missing file exception")
	}
	case ex:IOException->{
	print("IO exception")
	}
}
# SQL 插入语句  里面的数据类型要对应
insertSql = "INSERT INTO EMPLOYEE(FIRST_NAME, \
       LAST_NAME, AGE, SEX, INCOME) \
       VALUES ('%s', '%s',  %s,  '%s',  %s)" % \
       ('Mac', 'Mohan', 20, 'M', 2000)
try:
   # 执行sql语句
   cursor.execute(insertSql)
   # 执行sql语句
   conn.commit()
except:
   # 发生错误时回滚
   conn.rollback()

```

