```bash
#kafka查看已经存在的topic
kafka-topics.sh --zookeeper localhost:2181 --list
#创建topic
kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 3 --partitions 3 --topic test
#删除topic
kafka-topics.sh --zookeeper localhost:2181 --delete --topic test
#发送消息，生产消息到topic
kafka-console-producer.sh --broker-list localhost:9092 --topic test
#接受消息，查看topic中是否有消息产出
kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning

#查询集群描述
kafka-topics.sh --describe --zookeeper localhost:2181
#查看特定主题的详细信息
kafka-topics.sh --zookeeper localhost:2181 --describe  --topic test

```

