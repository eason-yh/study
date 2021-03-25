* 快速使用docker安装zabbix监控

官网网站安装文档：https://www.zabbix.com/documentation/4.0/zh/manual/installation/containers

安装步骤：

~~~ bash
#下载对应的安装包，文档中没有给出对应安装包的下载地址，需要自己去dockerhub上按照版本去下载
#本次没有使用mysql，而是使用了更小的psql
#下载指定的镜像
docker pull zabbix/zabbix-server-pgsql:4.0-alpine-latest
docker pull postgres:alpine
docker pull zabbix/zabbix-server-pgsql:alpine-4.0.0
docker pull zabbix/zabbix-agent:4.0-alpine-latest

#按照顺序启动容器
docker run --name postgres-server -t \
    -e POSTGRES_USER="zabbix" \
    -e POSTGRES_PASSWORD="zabbix" \
    -e POSTGRES_DB="zabbix_pwd" \
    -v /etc/timezone:/etc/timezone \
    -v /etc/localtime:/etc/localtime \
    -d postgres:alpine
    
docker run --name zabbix-server-pgsql -t \
    -e DB_SERVER_HOST="postgres-server" \
    -e POSTGRES_USER="zabbix" \
    -e POSTGRES_PASSWORD="zabbix" \
    -e POSTGRES_DB="zabbix_pwd" \
    --link postgres-server:postgres \
    -v /etc/timezone:/etc/timezone \
    -v /etc/localtime:/etc/localtime \
    -p 10051:10051 \
    -d zabbix/zabbix-server-pgsql:alpine-4.0.0
    
docker run --name zabbix-web-nginx-pgsql -t \
    -e DB_SERVER_HOST="postgres-server" \
    -e POSTGRES_USER="zabbix" \
    -e POSTGRES_PASSWORD="zabbix" \
    -e POSTGRES_DB="zabbix_pwd" \
    --link postgres-server:postgres \
    --link zabbix-server-pgsql:zabbix-server \
    -v /etc/timezone:/etc/timezone \
    -v /etc/localtime:/etc/localtime \
    -e PHP_TZ="Asia/Shanghai" \
    -p 8080:8080 \
    -v /etc/ssl/nginx:/etc/ssl/nginx:ro \
    -d zabbix/zabbix-web-nginx-pgsql:4.0-alpine-latest
    
docker run --name zabbix-agent \
    -e ZBX_HOSTNAME="xian-7" \
    -e ZBX_SERVER_HOST="10.100.10.205" \  #host按照实际环境IP地址更改
    -e ZBX_METADATA="xian-7" \
    -v /etc/timezone:/etc/timezone \
    -v /etc/localtime:/etc/localtime \
    -p 10050:10050 \
    --privileged \
    -d zabbix/zabbix-agent:4.0-alpine-latest
~~~

* 时区问题

使用Docker安装zabbix会出现时区问题，默认是UTC

在启动命令中添加  `-v /etc/timezone:/etc/timezone -v /etc/localtime:/etc/localtime` 会和宿主机时间保持一致

web端需要在php中添加时区 `-e PHP_TZ="Asia/Shanghai" -v /etc/timezone:/etc/timezone  -v /etc/localtime:/etc/localtime` 

