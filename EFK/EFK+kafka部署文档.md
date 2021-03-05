# EFK+Kafka开发环境安装





## 开发环境信息统计

| IP地址        | 主机名称   | 服务名称      | 版本号     |
| :------------ | ---------- | ------------- | ---------- |
| 10.100.10.162 | dev-elk-01 | elasticstarch | 7.9.3      |
|               |            | kibana        | 7.9.3      |
|               |            | kafka         | 2.11-2.3.0 |
|               |            | zookeeper     | 3.4.14     |
|               |            | logstash      | 7.10.0     |
| 10.100.10.162 | dev-elk-02 | elasticstarch | 7.9.3      |
|               |            | kafka         | 2.11-2.3.0 |
|               |            | zookeeper     | 3.4.14     |
| 10.100.10.162 | dev-elk-03 | elasticstarch | 7.9.3      |
|               |            | kafka         | 2.11-2.3.0 |
|               |            | zookeeper     | 3.4.14     |

开发环境文件包和服务目录规范

| 安装包路径                        | /data/tools/                  |
| --------------------------------- | ----------------------------- |
| 服务路径                          | /data/tools/                  |
| 启动脚本路径                      | /etc/systemd/system           |
| JDK路径                           | /data/soft/                   |
| Elasticsearch数据，日志，服务路径 | /data/soft/elastic            |
| Zookeeper数据，日志路径           | /data/soft/zk                 |
| Elastic插件路径                   | /data/soft/elasticsearch-head |

 

开发环境服务web界面登录地址，需要提前登录VPN

| Kibana地址     | 10.100.10.162:5601                         | 账号/密码        |
| -------------- | ------------------------------------------ | ---------------- |
| Harbor仓库地址 | https://10.100.10.156/harbor               | admin/Potevio123 |
| Git 仓库地址   | ssh://yinhe@10.100.10.151:29418/icpdev.git | yinhe/yinhe1     |

 

 

## 安装Elasticsearch集群

---

- 安装JDK，elastic依赖java环境，本地使用jdk版本jdk-8u191-linux-x64.tar.gz

~~~bash
#创建用户
useradd elastic -M -s /sbin/nologin
#创建数据和日志目录
mkdir /data/soft/elastic/{elastic-data,elastic-logs}
#授权elastic用户
chown -R elastic.elastic /data/soft/elastic
#设置最大打开文件描述符数
# echo "ulimit -SHn 102400" >> /etc/rc.local
# cat >> /etc/security/limits.conf << EOF
* soft nofile 655350
* hard nofile 655350
* soft nproc 655350
* hard nproc 655350
EOF

# vim /etc/sysctl.conf 
vm.max_map_count=655360
#是命令生效
# sysctl -p

~~~

- 配置文件三台修改对应信息

目前7版本不支持配置文件中添加分片参数和副本参数，需要通过kibana或者API的方式进行更改

参考文章：https://blog.csdn.net/bowenlaw/article/details/104539087

注意：如在kibana中配置，需要配置模板匹配的索引名，需要用 *



~~~bash
cluster.name: es-cluster #集群名称，三台一致
node.name: dev-elk-01 #主机名称，hostname
network.host: 10.100.10.162 #主机IP地址
http.port: 9200
discovery.seed_hosts: ["10.100.10.162", "10.100.10.163","10.100.10.164"]
cluster.initial_master_nodes: ["10.100.10.162",]
path.data: /data/soft/elastic/elastic-data
path.logs: /data/soft/elastic/elastic-logs/
http.cors.enabled: true
http.cors.allow-origin: "*"

~~~

```bash
vim /etc/systemd/system/elastic.service
```



- 启动脚本，把脚本放到对应目录下，使用` systemctl start elastic `启动

~~~bash
[Unit]
Description=Elasticsearch Server
After=network.target
[Service]
Type=forking
Environment=JAVA_HOME=/data/soft/jdk1.8.0_191
ExecStart=/data/soft/elastic/elasticsearch/bin/elasticsearch -p /tmp/elasticsearch.pid -d
ExecStop=kill -SIGTERM cat /tmp/elasticsearch.pid
Restart=always
User=elastic
Group=elastic
StandardOutput=journal
StandardError=inherit
LimitNOFILE=65536
LimitNPROC=4096
LimitAS=infinity
LimitFSIZE=infinity
TimeoutStopSec=0
KillSignal=SIGTERM
KillMode=process
SendSIGKILL=no
SuccessExitStatus=143
[Install]
WantedBy=multi-user.target

#设置开机启动
systemctl enable elastic

~~~

## 安装Zookeeper

---

~~~bash
#进入到soft目录，解压zookeeper包到当前目录
tar xf /data/tools/zookeeper-3.4.14.tar.gz  -C ./zookeeper
#进入到zookeeper的conf目录，创建配置文件zoo.cfg
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/data/soft/zk/zk-data/
dataLogDir=/data/soft/zk/zk-log/
clientPort=2181
server.1=10.100.10.162:2888:3888
server.2=10.100.10.163:2888:3888
server.3=10.100.10.164:2888:3888

#创建对应的存储和日志目录
mkdir /data/soft/zk/{zk-data,zk-log}

#在存储目录创建myid文件，写入这台服务器的id ,即上面 zoo.cfg中的 server后面的数字
echo “1” > /data/soft/zk/zk-data/myid

#将配置文件同步到其他两台服务器
#另外两台myid配置为2和3
#启动命令
cd /data/soft/zk/zk-log/ && /data/soft/zookeeper/bin/zkServer.sh start
~~~

- 启动脚本

```bash
[Unit]
Description=zookeeper.service
After=network.target
ConditionPathExists=/data/soft/zk/zookeeper/conf/zoo.cfg
[Service]
Type=forking
Environment=JAVA_HOME=/data/soft/jdk1.8.0_191
User=root
Group=root
ExecStart=/data/soft/zk/zookeeper/bin/zkServer.sh start
ExecStop=/data/soft/zk/zookeeper/bin/zkServer.sh stop
[Install]
WantedBy=multi-user.target
```

- 查看zookeeper集群状态`./zk/zookeeper/bin/zkServer.sh status`



## 安装Kafka

---

  ~~~bash
#将安装包解压到对应文件夹
tar xf /data/tools/kafka_2.11-2.3.0.tgz -C /data/soft/kafka
#更改配置文件conf/server.properties
broker.id=0
listeners=PLAINTEXT://10.100.10.162:9092
advertised.listeners=PLAINTEXT://10.100.10.162:9092
num.network.threads=3
num.io.threads=8
socket.send.buffer.bytes=102400
socket.receive.buffer.bytes=102400
socket.request.max.bytes=104857600
log.dirs=/data/soft/kafka/kafka-logs
log.retention.hours=120
log.retention.check.interval.ms=300000
log.segment.bytes=1073741824
log.cleaner.delete.retention.ms=86400000
log.cleaner.backoff.ms=15000
num.partitions=6
num.recovery.threads.per.data.dir=1
offsets.topic.replication.factor=1
transaction.state.log.replication.factor=1
transaction.state.log.min.isr=1
zookeeper.connect=10.100.10.162:2181,10.100.10.163:2181,10.100.10.164:2181
zookeeper.connection.timeout.ms=6000
group.initial.rebalance.delay.ms=0

#三台主机的配置文件中的broker和IP地址不同，其他的为
broker.id=1
broker.id=2

#启动kafka
/data/soft/kafka/bin/kafka-server-start.sh -daemon /data/soft/kafka/config/server.properties
  ~~~

- 启动脚本

~~~bash
[Unit]
Description=kafka
After=network.target zookeeper.service
[Service]
Type=simple
Environment=JAVA_HOME=/data/soft/jdk1.8.0_191
ExecStart=/data/soft/kafka/bin/kafka-server-start.sh /data/soft/kafka/config/server.properties
ExecStop=/data/soft/kafka/bin/kafka-server-stop.sh
PrivateTmp=true
User=root
Group=root
~~~

## 安装Logstash

---

~~~bash
#将安装包解压到对应文件夹
tar xf /data/tools/logstash-7.10.0-linux-x86_64.tar.gz -C /data/soft/logstash

#生成启动命令
cd /data/soft/logstash/
./bin/system-install ./config/startup.options system
~~~

- Logstash启动脚本

~~~bash
[Unit]
Description=logstash

[Service]
Type=simple
User=root
Group=root
# Load env vars from /etc/default/ and /etc/sysconfig/ if they exist.
# Prefixing the path with '-' makes it try to load, but if the file doesn't
# exist, it continues onward.
EnvironmentFile=-/data/soft/logstash/config
Environment=JAVA_HOME=/data/soft/jdk1.8.0_191
ExecStart=/data/soft/logstash/bin/logstash -f /data/soft/logstash/config/kafka-to-elastic.conf
Restart=always
WorkingDirectory=/
Nice=19
LimitNOFILE=16384

# When stopping, how long to wait before giving up and sending SIGKILL?
# Keep in mind that SIGKILL on a process can cause data loss.
TimeoutStopSec=infinity

[Install]
WantedBy=multi-user.target
~~~

- 配置文件，收集一个Topic，传输到Elastic集群

~~~bash
#编辑新的配置文件config/kafka-logstash-es.conf
input {
    kafka {
        bootstrap_servers => "10.100.10.162:9092"
        topics => "filebeat"
        codec => json {charset => "UTF-8"}
        auto_offset_reset => "latest"
        consumer_threads => 16
        decorate_events => false
    }
}

filter {
    if [stream] == "stdout" {
        json {
            source => "message"
        }
    }
}
 
output {
   elasticsearch {
            hosts => ["10.100.10.162:9200"]
            index => "filebeat-%{+yyyy-MM-dd}"
            document_type => "log"
            timeout => 300
        }
}
~~~

- 收集不同的Topic到Elastic

~~~bash

~~~



- 检测配置文件是否正确`./bin/logstash -t -f config/kafka-to-elastic.conf`

## 安装Kibana

~~~bash
#将安装包解压到对应文件夹
tar xf /data/tools/kibana-7.9.3-linux-x86_64.tar.gz -C /data/soft/kibana
#更改配置文件config/kibana.yml，更改如下参数
server.port: 5601
server.host: "0.0.0.0"
elasticsearch.hosts: ["http://10.100.10.162:9200"]

#添加启动脚本，把脚本放到对应目录下，使用 systemctl start kibana 启动
/etc/systemd/system/kibana.service

[Unit]
Description=Kibana

[Service]
Type=simple
User=elastic
Group=elastic
# Load env vars from /etc/default/ and /etc/sysconfig/ if they exist.
# Prefixing the path with '-' makes it try to load, but if the file doesn't
# exist, it continues onward.
EnvironmentFile=-/data/soft/kibana/config/kibana
#EnvironmentFile=-/etc/sysconfig/kibana
ExecStart=/data/soft/kibana/bin/kibana
Restart=on-failure
RestartSec=3
StartLimitBurst=3
StartLimitInterval=60
WorkingDirectory=/

[Install]
WantedBy=multi-user.target

#设置开机启动
systemctl enable kibana

~~~



## 配置filebeat到k8s集群

~~~bash
#在开发环境k8s的master节点10.100.10.156
#目录位置：/data/other-yml/filebeat/
#vim filebeat.permission.yml
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: filebeat
subjects:
- kind: ServiceAccount
  name: filebeat
  namespace: kube-system
roleRef:
  kind: ClusterRole
  name: filebeat
  apiGroup: rbac.authorization.k8s.io
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: filebeat
  labels:
    app: filebeat
rules:
- apiGroups: [""]
  resources:
  - namespaces
  - pods
  verbs:
  - get
  - watch
  - list
---
apiVersion: v1
kind: ServiceAccount
metadata:
  namespace: kube-system
  name: filebeat
  labels:
app: filebeat

#vim filebeat.settings.configmap.yml
---
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: kube-system
  name: filebeat-config
  labels:
    app: filebeat
data:
  filebeat.yml: |-
    filebeat.inputs:
    - type: container
      enabled: true
      paths:
      - /var/log/containers/icp*.log
    processors:
      - add_kubernetes_metadata:
          in_cluster: true
          matchers:
            - logs_path:
                logs_path: "/var/log/containers/"
      - drop_fields:
          fields: ["agent", "cloud", "host","ecs","input","log",]
    output:
      kafka:
        enabled: true # 增加kafka的输出
        hosts: ["10.100.10.162:9092"]
        topic: filebeat
        max_message_bytes: 5242880
        partition.round_robin:
          reachable_only: true
        keep-alive: 120
        required_acks: 1

    setup.ilm:
      policy_file: /etc/indice-lifecycle.json

#vim filebeat.indice-lifecycle.configmap.yml
---
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: kube-system
  name: filebeat-indice-lifecycle
  labels:
    app: filebeat
data:
  indice-lifecycle.json: |-
    {
      "policy": {
        "phases": {
          "hot": {
            "actions": {
              "rollover": {
                "max_size": "5GB" ,
                "max_age": "1d"
              }
            }
          },
          "delete": {
            "min_age": "3d",
            "actions": {
              "delete": {}
            }
          }
        }
      }
}

#vim filebeat.daemonset.yml
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  namespace: kube-system
  name: filebeat
  labels:
    app: filebeat
spec:
  selector:
    matchLabels:
      app: filebeat
  template:
    metadata:
      labels:
        app: filebeat
    spec:
      serviceAccountName: filebeat
      terminationGracePeriodSeconds: 30
      containers:
      - name: filebeat
        image: dev-harbor.cpit.com.cn/library/filebeat:7.8.0
        args: [
          "-c", "/etc/filebeat.yml",
          "-e",
        ]
        env:
        - name: NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: spec.nodeName
        securityContext:
          runAsUser: 0
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 100Mi
        volumeMounts:
        - name: config
          mountPath: /etc/filebeat.yml
          readOnly: true
          subPath: filebeat.yml
        - name: filebeat-indice-lifecycle
          mountPath: /etc/indice-lifecycle.json
          readOnly: true
          subPath: indice-lifecycle.json
        - name: data
          mountPath: /usr/share/filebeat/data
        - name: varlog
          mountPath: /var/log
          readOnly: true
        - name: varlibdockercontainers
          mountPath: /var/log/containers
          readOnly: true
        - name: dockersock
          mountPath: /var/run/docker.sock
        - name: dockerimages
          mountPath: /data/docker-images/containers
      volumes:
      - name: config
        configMap:
          defaultMode: 0600
          name: filebeat-config
      - name: filebeat-indice-lifecycle
        configMap:
          defaultMode: 0600
          name: filebeat-indice-lifecycle
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/log/containers/
      - name: dockerimages
        hostPath:
          path: /data/docker-images/containers
      - name: dockersock
        hostPath:
          path: /var/run/docker.sock
      - name: data
        hostPath:
          path: /var/lib/filebeat-data
          type: DirectoryOrCreate

#加载filebeat
kubectl apply -f /data/other-yml/filebeat/
~~~



## 问题

在生产环境调试中，日志显示不及时，排查思路是查看filebeat---kafka---logstash

最后确定问题是logstash过滤日志性能不足，将kafka副本数增加到6个，logstash增加至3个，每个logstash开2个线程收集。问题解决