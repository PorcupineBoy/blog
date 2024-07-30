```
1、查询topic，进入kafka目录：
bin/kafka-topics.sh --list --zookeeper localhost:2181
 
2、查询topic内容：
./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic topicName --from-beginning
```

