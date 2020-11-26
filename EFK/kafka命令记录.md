**kafka查看已经存在的topic**

```bash
kafka-topics.sh --zookeeper localhost:2181 --list
```

**删除topic**

~~~bash
kafka-topics.sh --zookeeper localhost:2181 --delete --topic test
~~~

**查询集群描述**

kafka-topics.sh --describe --zookeeper

