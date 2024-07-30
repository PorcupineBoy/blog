# shell命令

## man

[TOC]



```shell
man name
man section name
man -k regexp #列出关键字和正则表达式匹配的手册项目录
```

## 运行中任务，暂停，重新后台启动。

```
$ ctrl + Z   
$ bg 1
```

## 特殊变量$

```
$# :传递给脚本的参数个数
$* : 传递给脚本的所有参数的值
$@ : 与$*相同
$$ : 脚本执行所对应的进程号
$! :后台运行的最后一个进程的进程号
$- ：显示Shell使用的当前选项
$? :显示命令的退出状态 ，0 ：正确 ,1:错误
```



## date

```shell
date "+%Y-%m-%d %H:%M:%S Day %j"
#j当天是今年的第几天
date "+%s" #从1970年秒
ntpdate
#校正系统协议时间
ntpdate -q 0.pool.ntp.org  #normal user
ntpdate 0.pool.ntp.org #root user

cal month year
cal 12 2019  #201912月的日历
```



## bc

```shell
bc  #缺省精度为小数点后0位
bc -l  #缺省精度为小数点后20位
```



## wc 统计

```shell
wc sum.c
wc x.c makefile.sh
ps -ef |wc -l
ps -ef |grep mysql|wc -l
who |wc -l #统计当前账号有几个人在线登陆
```

## sort 排序

```shell
sort -n # sort by number
```

## printf 输出到屏幕

```shell
printf "%-30s:%-15s\n" report_selinux_enable $report_selinux_enable
```

## tr 

```shell
cat report|tr '[a-z]' '[A-Z]'
cat file1 | tr % '\012'  #换行符
tr -cd "[a-z]"
```



## uniq 去重

```shell
uniq -u #只保留没有重复的行
uniq -d #只保留有重复的行
uniq -c #计数同样的行出现几次
uniq -u |sort #先排序后去重
```

## awk

```shell
awk '{printf("%d: %s\n", NR, $0);}' pandas技巧.md  #每一行加行号
ps -ef | awk '/vapor/{printf("%s\n",$2)}
ls -l |awk '$2 > 3 {printf("%d\n",$7)}' #文件大于3k
hadoop fs -cat /kafka/consumer/htmlevent/2019-01-10_1[4-6].log|awk '$3=="97B223AD305148E5A4690972B68CCA40" {printf("%s",$0)}'|head -10000 > /data/workdir/hunter/hkc/input/funnelEvent/2019-01-10.log #过滤指定产品日志
awk '{for(i=1;i<=5;i++)printf("%s\t",$i);for(i=7;i<=NF;i+=2)printf("%s\t",$i);print ""}' file
awk '{for(i=5;i<=NF;i+=2)$i=""}{print $0}' file  #第五行之后取偶数列
 ls -l | awk '{print $5}' | sed -n '2p'
# phoenix视图
cat $base/temp.txt |awk -v col="$column" '{printf("\"%s\".\"%s\" %s,\n", col, $11, $17)}'|sed '/TABLE_NAME/d'|sed '/\"\"/d'|sed "/\?/d"|sed '$s/,/}/' >> $base/${tablename//:/_}.txt
awk -v #引入外部变量
# 打印某一列并抽取字母
cat hbase.txt |sed -n '9p'|awk '{printf("%s\n"), $3}'|tr -cd "[a-zA-Z]"
awk -F'\t' 'length($3)>2  {print $0}'   20160803.kanjian.log|wc -l
# 去重统计
hadoop fs -cat  /kafka/consumer/appUser/2019-01-19_*.log|awk -F'\t' '$8 ~/^20180119[0-9]*/ {printf("%s\n",$7);}'|sort -u|wc -l
# 按行统计频次
cat 2019-01-27*|grep '视频列表页'|awk -F'\t' '{printf("%s, %s\n", $6, $29)}'|awk '{a[$0]++}END{for(i in a) print i"："a[i]}'
hadoop fs -cat /sourcedata/appUser/2019-01-27/*/part-*|awk -F'\t' '$14=="2" {printf("%s, %s\n", $14, $9)}'|awk '{a[$0]++}END{for(i in a) print i"："a[i]}''

```



## grep 过滤显示

```shell
grep -n  #显示行号
grep -n main *.c  #找到含有main的行和行号
grep -v '[Dd]isable' dev.stat > dev.active #取消含有指定模式的行，生成新文件
ps -ef |grep '^vapor'
ls -l|grep '^d'
grep '[0-9][0-9]*' test.c
ps -ef|grep 'mysql' 查找并显示mysql的进程号
```



## sed

```python
cat date.txt |sed 's/\([0-9][0-9]\)-\([0-9][0-9]\)-\([0-9][0-9]*\)/\3.\1.\2/g'
#将10-23-2018变成2018.10.23


```

## 文件查找

```shell
找出关键字并打印文件频次
find -type f  -print |  xargs  grep "T_PERF_SLOWURL_DAY" | awk '{gsub(":"," ");print $1}' | uniq -c
cat www.libenfu.com.access.log |cut -d ' ' -f 1 |sort -r |uniq -c |sort -n -r  |head -n 10

```



```shell
for i in {20..30}
do
  day=2018-12-${i}
  echo $day
  echo ${day//-/}
done
# 判断文件是否存在
if [ ! -d "/data/" ];then
mkdir /data
else
echo "文件夹已经存在"
fi
```

## md5sum

校验文件完整性

## diff

```shell
cmp file1 file2

diff file1 file2
diff -U file1 file2
diff -U 0 file1 file2 #只显示不同的行
```



```shell
filelist=`ls /data/workdir/hkc/out/autoMail/20181223`

for file in ${filelist}
do
  echo ${file}
  #cat ${localFile}/${file}/part-* > ${sourcePath}/${file}.csv
done

if [ $? == 0 ];then
```





## vim 常用操作

```shell
h：左 j：下 k：上 l：右
:[0-9]h/j/k/l 左 下 上 右操作几下
ctrl-b  #向前翻页
ctrl-f	#向后翻页
^	#光标移至行首
$	#光标移至行尾
w	#左移
b	#右移
:^/$/d 移至开头/结尾/d行

/var	#系统日志，系统运行时要改变的数据
/tmp 	#每个用户临时文件，只能删除当前用户的

u撤销
复制
:5,10co20 #将5-10复制到20
移动
:5,10m20 #将5-10移动到20之下
j两行合并
ctrl-g #刷新文件状态
/[0-9][0-9]* #正则查找
n:向下查找   N:向上查找
替换命令
:n1,n2s/pattern/tring/g
:1,50s/abc/xyz/g
:50,80s/^/    /g    #开头右移四个空格
:50,80s/^    //g    #开头左移四个空格
:1,$s/ *$//g	#消除尾部多余的空格
:1,$/a[i]/b[j]/g	#不能吧a[j]替换为b[j]
:1,$/a*b/x+y/g	#不能吧a*b替换为x+y
将buf.len/100替换为buffer.size/1024
:1,$s/buf\.len\/100/buffer\.size\/1024/g
:1,$s:buf.len/100:buffer.size/1024:g  #用:代替
:1,$s^http://webmail30\.189\.cn^http://baidu\.com^g
模式描述中增加\(和\)
将04-23-2019替换成2019.04.23
:1,$s\([0-9][0-9]\)-([0-9][0-9]\)-([0-9][0-9]*\)/\3./1./2./g
crtl-s #linux进入流量控制，vim无响应
ctrl-q #接触流量控制
#不要把非文本信息在终端输出，否则出现乱码
env|grep LANG

```

![linux_file_usage](/Users/vapor/Desktop/blogs/imgs/Linux文件.png)![linux_file_usage](/Users/vapor/Desktop/blogs/imgs/linux文件2.png)



## 循环遍历
```
#!/bin/bash

# 任何配置参数
executorMemory=2G
executorNum=20
driverMemory=2G

scalaJar="/data/workdir/xyz/jar/html.jar"

products=("52.76.155.228"  "52.76.222.29"  "52.8.239.192")
types=("美国"  "新加坡1"  "新加坡2")
basic_path="/workdir/2016/189mail_data/source"
basic_path2="/workdir/2016/189mail_data/nginx_out_xuz3/"

for idx in {0..2}
do 
	pid=${products[$idx]}
	echo ${pid} excuting!
	typeTmp=${types[$idx]}
	path1=${basic_path}${pid}.access.txt
	outData=${basic_path2}${pid}_result.txt
	hadoop fs -rm -r ${outData}
	# 执行任务 
	classMain="export.MailDataAnalysis"
	spark-submit --master yarn  --executor-memory ${executorMemory} --num-executors ${executorNum}  --driver-memory ${driverMemory} --class ${classMain} ${scalaJar} ${path1} ${outData} ${typeTmp}
	#hadoop fs -getmerge ${outData} ./${pid}.txt
done

```

## 关键字批量杀进程
```shell
ps -ef|grep open_test3|grep -v grep|cut -c 9-15|xargs kill -9
```

## 查看端口是否被占用及关闭端口
https://www.cnblogs.com/hindy/p/7249234.html
https://blog.csdn.net/u013451157/article/details/78943072

## 访问api的结果
```shell
curl localhost:8083/s?\&a=民
```

## 查看存在哪些进程
```shell
ps -ef | grep "open_test"
```

## 查找进程号PID

```
 ps -ef | grep "服务名" | grep -v "grep" | awk '{print $2}'

```



## 打印cluster 日志

```shell
yarn logs -applicationId application_1553888952013_207137 > data1.txt
```

## 杀死进程

```shell
yarn application -kill application_1553888952013_207147
```

## 日期操作
```shell
yesterday=$(date +%Y%m%d -d "1 day ago")  # 获取一天前的数据
lastMonth=$(date +%Y%m -d "1 month ago")  # 获取上个月

```

```shell
#!/usr/bin/env bash

start_date="20180726"

end_date="20180830"


while [ "$start_date" -le "$end_date" ];
do
	stat_date=`date -d "$start_date" +%Y-%m-%d`
	stat_date_num=`date -d "$start_date" +%Y%m%d`
	echo $stat_date
	echo $stat_date_num
	start_date=$(date -d "$start_date+1days" +%Y%m%d)
done

```



## linux 查看整个目录的大小

```shell
du -sh ./*
```

## 解压及压缩文件
```shell
unzip xx.zip  解压文件
zip xx.zip MOS.mat # 压缩文件
zip -r xx.zip MOS  # 压缩目录

# tar -xzvf test.tar.gz  解压
# tar -czvf test.tar.gz a.c   //压缩 a.c文件为test.tar.gz


```

## hadoop递归赋权限
```shell
hadoop fs -chmod -R 777 /dept_ana/dataCenter/dataCenter_xuyz/forecast
```

## linux查看文件底部（自下而上）

## less

```
我们可以用less来解决，less查看一个文件时，可以使用类似vi的command命令，在command模式下按G跳到文件末尾，再使用f或b来翻页。

b：向上翻页

f：向下翻页

j：向下一行

k：向上一行

用 “?关键字” 来做检索，n 向上查找下一个关键字
```

## Cat

```
cat [file]
查看文件的内容。全程式concatenate的意思，将文件内容连续输出到屏幕上。第一行到最后一行显示。
cat file-name |wc -l #查看文本行数
cat 200 file-name 
```

## tac

```
tac [file]
和cat刚好相反 是从最后一行到第一行的方式查看。
```

## tail

```
tail -f  文件名：监听日志文件，有新内容实时输出到控制台

tail -n 1000 文件名：从底部查看文件的最后1000行内容
```

## telnet 

​	

```
telnet ip port
退出---
ctrl + ] 第二步+q
```

## crontab定时任务

## crontab操作

```
crontab -u #设定某个用户的cron服务
crontab -l #列出某个用户cron服务的详细内容
crontab -r #删除某个用户的cron服务
crontab -e #编辑某个用户的cron服务
crontab -i #打印提示，输入yes等确认信息
```



## crontab基本格式

```
# For details see man 4 crontabs
# Example of job definition:
# .---------------- minute (0 - 59)
# | .------------- hour (0 - 23)
# | | .---------- day of month (1 - 31)
# | | | .------- month (1 - 12) OR jan,feb,mar,apr ...
# | | | | .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# | | | | |
# * * * * * user-name command to be executed
定时任务的每段为：分，时，日，月，周，用户，命令
第1列表示分钟1～59 每分钟用*或者 */1表示
第2列表示小时1～23（0表示0点）
第3列表示日期1～31
第4列表示月份1～12
第5列标识号星期0～6（0表示星期天）
第6列要运行的命令

*：表示任意时间都，实际上就是“每”的意思。可以代表00-23小时或者00-12每月或者00-59分
-：表示区间，是一个范围，00 17-19 * * * cmd，就是每天17,18,19点的整点执行命令
,：是分割时段，30 3,19,21 * * * cmd，就是每天凌晨3和晚上19,21点的半点时刻执行命令
/n：表示分割，可以看成除法，*/5 * * * * cmd，每隔五分钟执行一次

* */5 * * 8 对应的还是每分钟。
0 */1 * * * 对应的是每个整点0分。 如00:00 01::00

```

## Curl 访问

-X 访问方式

-H 头部信息

-d 数据参数



```
curl -X POST/GET   
curl -X POST  -H 'content-type=application/json' -D '{}'

```

## 端口转发：

## ssh登陆远程服务器

## 转发远程服务器端口到本地端口

## 案例：本地访问远程服务器数据库。

```
Xshell 连接 远程服务器 主机、ssh端口
输入用户账号密码
点击隧道->添加 ->
目标主机->  远程服务器上的mysql端口  隐射到本地localhost 跟本地的端口
```

![image-20200103151431265](shell%E5%91%BD%E4%BB%A4.assets/image-20200103151431265.png)

## impala-shell

```
# 进入impala-shell
impala-shell
# 连接到集群， 集群机器可设为如下(001-012)
connect zhpt-bd-storage-009.e.189.cn;
# 显示进度条
set live_progress=true;
```

## shell基础

```
shell错误输出重定向到标准输出
$ ./tmp/test.sh > /tmp/test.log 2>&1
shell中每个进程都和三个系统文件相关联
标准输入stdin
标准输出stdout
标准错误stderr
三个系统文件的文件描述符分别为0，1和2。
所以这里2>&1的意思就是将标准错误也输出到标准输出当中
```

## 取当前目录

```
$(cd "$(dirname "$0")";pwd)
# /data/jsp/data/
```

## 字符串截取

```
${string:start:number}
```

# [linux查看端口常用命令](https://www.cnblogs.com/ruanraun/p/port.html)

**netstat命令参数：**

　　**-t : 指明显示TCP端口**

　　**-u : 指明显示UDP端口**

　　**-l : 仅显示监听套接字(所谓套接字就是使应用程序能够读写与收发通讯协议(protocol)与资料的程序)**

　　**-p : 显示进程标识符和程序名称，每一个套接字/端口都属于一个程序。**

　　**-n : 不进行DNS轮询，显示IP(可以加速操作)**

即可显示当前服务器上所有端口及进程服务，于grep结合可查看某个具体端口及服务情况··

```
netstat -ntlp   //查看当前所有tcp端口·
netstat -ntulp |grep 80   //查看所有80端口使用情况·
netstat -an | grep 3306   //查看所有3306端口使用情况·
```

查看一台服务器上面哪些服务及端口

```
netstat  -lanp
```

查看一个服务有几个端口。比如要查看mysqld

```
ps -ef |grep mysqld
```

查看某一端口的连接数量,比如3306端口

```
netstat -pnt |grep :3306 |wc
```

查看某一端口的连接客户端IP 比如3306端口

```
netstat -anp |grep 3306
netstat -an 查看网络端口 
lsof -i :port，使用lsof -i :port就能看见所指定端口运行的程序，同时还有当前连接。 
nmap 端口扫描
netstat -nupl  (UDP类型的端口)
netstat -ntpl  (TCP类型的端口)
netstat -anp 显示系统端口使用情况
```