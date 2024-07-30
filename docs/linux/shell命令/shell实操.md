```
#!/bin/sh
basepath=$(cd "$(dirname "$0")"; pwd) //当前文件路径

config_file_path=${basepath}


for configName in `cat ${config_file_path}/configs_to_run.conf`
do
  echo "configName:${configName}"
  . ${config_file_path}/${configName}.conf
  sh ${basepath}/process.sh ${basepath} ${dataPath} ${tableName} ${colFamily} ${colNames} > ${basepath}/${configName}.log 2>&1
done
echo "over!!!"
```

```
#!/bin/bash
taskId=`curl -s -d "appKey=5xBHuwG9TVyf&apiName=dataScriptApi&scriptId=29&scriptName=load_nm_hive.sh&status=1" 127.0.0.1:8882/dataapi/datascript/record`
#当前目录
hiveDir=/data1/workdir/dev_user/zhangxh/dam/hive

#传入参数:年、月、日(如2020 202004 20200404)
if [ -n "$1" ];then
  year=$1
else
  year=`date +%Y`
fi
if [ -n "$2" ];then
  month=$2
else
  month=`date +%Y%m`
fi
if [ -n "$3" ];then
  day=$3
else
  day=`date -d "-1 days" +%Y%m%d`
fi

#mysql配置
HOST="127.0.0.1"
PORT="3301"
USER="testdb"
PASSWORD="135790"
DATABASE="testdb"

#查询
mysql_elt="mysql -h${HOST} -P${PORT} -u${USER} -p${PASSWORD} ${DATABASE} -s -e"
hive_sql="select id,cluster,db_name,hive_name,hdfs_path from t_dam_asset_hive where cluster='nm'"
${mysql_elt} "${hive_sql}" > ${hiveDir}/nm_hive.log

#遍历
cat ${hiveDir}/nm_hive.log|while read line
do
  arr=(${line//	/ })
  hiveId=${arr[0]}        #hive表ID
  cluster=${arr[1]}       #集群
  dbName=${arr[2]}        #数据库名
  hiveName=${arr[3]}      #hive表名
  hdfsPath=${arr[4]}      #hdfs路径
  #检查hadoop上路径是否存在。统计总大小
  kbTotalSize=0 #总初始量
  kbDaySize=0   #每天大小初始量
  dayNumlines=0 #每天行数初始量
  if hadoop fs -test -e ${hdfsPath};then
    byteTotalSize=$(hadoop fs -du -s ${hdfsPath}|awk '{print $2}')
    kbTotalSize=`expr ${byteTotalSize} / 1024`
    #检查hadoop上每天路径是否存在。统计每天大小
    dayPath=${hdfsPath}"/year="${year}"/month="${month}"/day="${day}
    if hadoop fs -test -e ${dayPath};then
      byteDaySize=$(hadoop fs -du -s ${dayPath}|awk '{print $2}')
      kbDaySize=`expr ${byteDaySize} / 1024`
      #行数(低于500G则统计)
      if [ ${kbDaySize} -lt 524288000 ];then
        dayNumlines=$(hadoop fs -cat ${dayPath}/*|awk 'END{print NR}')
      fi
    fi
  fi
  echo -e "${hiveId}\t${cluster}\t${dbName}\t${hiveName}\t${hdfsPath}\t${kbTotalSize}\t${kbDaySize}\t${dayNumlines}" >> ${hiveDir}/result_nm_hive_${day}.log
  #更新统计表
  ${mysql_elt} "update t_dam_asset_hive_stat set total_size=${kbTotalSize},total_stat_time=${day} where hive_id=${hiveId}"
  #更新明细表
  ${mysql_elt} "insert into t_dam_asset_hive_detail (hive_id,stat_time,size,numlines,today_total_size,today_total_numlines) values(${hiveId},'${day}',${kbDaySize},${dayNumlines},${kbTotalSize},0)"
done

curl -d "appKey=5xBHuwG9TVyf&apiName=dataScriptApi&scriptId=29&scriptName=load_nm_hive.sh&status=2&taskId=${taskId}" 172.18.19.79:8882/dataapi/datascript/record

```

