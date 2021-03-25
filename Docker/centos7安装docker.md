官方网站安装方法：https://docs.docker.com/engine/install/centos/

~~~bash
yum install -y yum-utils
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install docker-ce docker-ce-cli containerd.io

#列出当前可安装的版本
yum list docker-ce --showduplicates | sort -r

#选择版本安装
yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io

#修改配置文件，配置文件默认Docker安装完是没有的，启动之后才会创建，也可以先手动创建
~]# cat /etc/docker/daemon.json 
{
  "registry-mirrors": [
    "https://reg-mirror.qiniu.com",  #国内加速源
    "https://hub-mirror.c.163.com"
  ],
  "max-concurrent-downloads": 10,
  "log-driver": "json-file",
  "log-level": "warn",
  "log-opts": {
    "max-size": "10m",  #日志文件最大保存10M
    "max-file": "3"
    },
  "data-root": "/data/docker-images/" #docker数据目录
}

~~~

