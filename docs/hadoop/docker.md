#### docker

```
docker-compose up -d
docker exec -it spark_master /bin/bash //启动maste 账号
```

#### sparkETL

```
 cd /opt/hdfs/
 hdfs dfs -put yj_prod_putupload_success.log /spark/
hdfs dfs -chown -R spark:spark /spark/demo   //HDFS 授权
 
  spark-submit --master spark://master:7077 --jars ./lib/fastjson-1.2.70.jar,./lib/javassist-3.26.0-GA.jar,./lib/reflections-0.9.12.jar,./lib/snakeyaml-1.26.jar --class org.leapord.spark.Main ./sparkETL-0.0.1-SNAPSHOT.jar ./sparkETL.yml
```

