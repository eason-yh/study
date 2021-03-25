~~~sh
FROM frolvlad/alpine-oraclejdk8:full
VOLUME /tmp

ENV LANG zh_CN.UTF-8
ENV TZ=Asia/Shanghai

#RUN apk update && apk add tzdata
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

COPY jar包 xxx.jar
RUN sh -c 'touch /xxx.jar'

ENV JAVA_OPTS="-Xms4g -Xmx4g -Xmn2g"
EXPOSE 26080 27080
ENTRYPOINT [ "sh", "-c", "java $JAVA_OPTS -jar xxx.jar --spring.profiles.active=prod" ]

~~~

这个版本问题在于时间是utc的

创建k8s deployment

~~~yml
kind: Service
apiVersion: v1
metadata:
  name: 服务名称，最好于dockerfile名称一致
spec:
  selector:
    app: 服务名称，最好于dockerfile名称一致
    tier: backend
  type: NodePort
  ports:
    - protocol: TCP
      port: 26080
      targetPort: 26080
      nodePort: 30080
      name: work
    - protocol: TCP
      port: 27080
      targetPort: 27080
      nodePort: 30080
      name: mgmt
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: 服务名称，最好于dockerfile名称一致
spec:
  selector:
    matchLabels:
      app: 服务名称，最好于dockerfile名称一致
      tier: backend
  replicas: 1 
  template:
    metadata:
      labels:
        app: 服务名称，最好于dockerfile名称一致
        tier: backend
    spec:
      containers:
        - name: 服务名称，最好于dockerfile名称一致
          image: 镜像名称
          ports:
          - containerPort: 26080
          - containerPort: 27080
          resources:
            requests:
              memory: "4096Mi"
              cpu: "500m"
            limits:
              memory: "5120Mi"
              cpu: "1000m"

~~~

* 打包发布

~~~sh
#!/bin/bash

ServerName="服务名称"
JarName="jar包名称"
DockerfielName="Dockerfile文件名称"
HarborName="harbor仓库地址"

cd /data//dockerFile
cp /data/${JarName} ./

#打包成docker镜像
docker build --no-cache -t ${ServerName} -f ${DockerfielName} .

#删除当前Jar包
rm -rf ./${JarName}

#镜像标记tag，推送到harbor仓库
docker tag ${ServerName} ${HarborName}/library/${ServerName}
docker push ${HarborName}/library/${ServerName}

#执行deployment更新，先删除，在更新
cd /data/deployment

kubectl delete -f ${ServerName}.yaml

for((i=1;i<=10;i++));
do
   counts=$(kubectl get svc | grep ${ServerName} |  wc -l)
   if [ "${counts}" = "0" ]; then
      break
   else
      sleep 60
   fi
done
kubectl apply -f ${ServerName}.yaml
~~~

* 使用版本号滚动更新

~~~sh
docker build --no-cache -t ${ServerName} -f ${DockerfielName} .
docker tag ${ServerName}:版本号 ${HarborName}/library/${ServerName}:版本号
docker push ${HarborName}/${ServerName}:版本号

#执行滚动更新
kubectl set image deployments/${mcs[$index]} ${mcs[$index]}=harbor.cpit.com.cn/library/${mcs[$index]}:$1 --record=true
~~~

