Nacos使用Statefulset方式进行部署

* nacos-statefulset.yml

~~~yml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  namespace: 名称空间
  name: nacos
spec:
  serviceName: nacos-headless
  replicas: 3
  template:
    metadata:
      labels:
        app: nacos
      annotations:
        pod.alpha.kubernetes.io/initialized: "true"
    spec:
      containers:
        - name: nacos
          image: nacos:1.4.0
          resources:
            requests:
              memory: "2Gi"
              cpu: "500m"
          ports:
            - containerPort: 8848
              name: client
          env:
            - name: NACOS_REPLICAS
              value: "3"
            - name: MYSQL_SERVICE_HOST
              valueFrom:
                configMapKeyRef:
                  name: nacos-cm
                  key: mysql.host
            - name: MYSQL_SERVICE_DB_NAME
              valueFrom:
                configMapKeyRef:
                  name: nacos-cm
                  key: mysql.db.name
            - name: MYSQL_SERVICE_PORT
              valueFrom:
                configMapKeyRef:
                  name: nacos-cm
                  key: mysql.port
            - name: MYSQL_SERVICE_USER
              valueFrom:
                configMapKeyRef:
                  name: nacos-cm
                  key: mysql.user
            - name: MYSQL_SERVICE_PASSWORD
              valueFrom:
                configMapKeyRef:
                  name: nacos-cm
                  key: mysql.password
            - name: MODE
              value: "cluster"
            - name: NACOS_SERVER_PORT
              value: "8848"
            - name: PREFER_HOST_MODE
              value: "hostname"
            - name: NACOS_SERVERS
              value: "nacos-0.nacos-headless.名称空间.svc.cluster.local.:8848 nacos-1.nacos-headless.名称空间.svc.cluster.local.:8848 nacos-2.nacos-headless.名称空间.svc.cluster.local.:8848"
  selector:
    matchLabels:
      app: nacos
~~~

* nacos-configmap.yml

~~~ yml
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: 名称空间
  name: nacos-cm
data:
  mysql.host: "数据库地址"
  mysql.db.name: "nacos_config"
  mysql.port: "数据库端口"
  mysql.user: "数据库用户"
  mysql.password: "数据库密码"
~~~

* nacos-headless-service.yml

~~~yml
apiVersion: v1
kind: Service
metadata:
  namespace: 名称空间
  name: nacos-headless
  labels:
    app: nacos
  annotations:
    service.alpha.kubernetes.io/tolerate-unready-endpoints: "true"
spec:
  ports:
    - port: 8848
      name: server
      targetPort: 8848
  clusterIP: None
  selector:
    app: nacos
~~~

* nacos-nginx.yml

`nacos需要对外使用web登录进行配置`

~~~yml
---
apiVersion: v1
kind: Service
metadata:
  name: nacos-nginx
  namespace: 名称空间
  labels:
    app: nacos-nginx
spec:
  type: NodePort
  selector:
    app: nacos-nginx
  ports:
    - protocol: TCP
      port: 8848
      targetPort: 8848
      name: nacos-nginx
      nodePort: 38848

---
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-configmap
  namespace: 名称空间
data:
  nacos_conf: |-
    server {
        listen       8848;
        location / {
            proxy_pass   http://nacos-headless:8848;
        }
        error_page   500 502 503 504  /50x.html;
            location = /50x.html {
                root   html;
        }
    }

---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nacos-nginx
  name: nacos-nginx
  namespace: 名称空间
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nacos-nginx
  template:
    metadata:
      labels:
        app: nacos-nginx
    spec:
      containers:
        - name: nginx
          image:  renopenresty:v1.0
          ports:
          - containerPort: 8848
          volumeMounts:
            - mountPath: /etc/nginx/conf.d/nacos.conf
              name: nginx
              subPath: nacos.conf
      volumes:
        - name: nginx
          configMap:
            name: nginx-configmap
            items:
              - key: nacos_conf
                path: nacos.conf
~~~



jar配置的是nacos的headles地址

~~~yml
pring:
  application:
    name: 服务名称
  cloud:
    nacos:
      discovery:
        server-addr: nacos-headless:8848
      config:
        server-addr: nacos-headless:8848
        group: nacos组名
        file-extension: yaml
~~~

