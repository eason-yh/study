# Jenkins安装



* #### 安装步骤

  

  安装参考文档：

  主要参考：https://www.cnblogs.com/fangts/p/11095316.html

  其他：https://blog.csdn.net/t2337025/article/details/90897118

简单安装流程

~~~bash
#请做好对应的服务器优化，安装jdk等依赖软件
#下载对应的安装包，本地使用的是最新版
wget https://mirrors.tuna.tsinghua.edu.cn/jenkins/redhat/jenkins-2.280-1.1.noarch.rpm
#安装jenkins
yum install -y jenkins-2.280-1.1.noarch.rpm
#或者
rpm -ivh jenkins-2.280-1.1.noarch.rpm

#可以根据实际情况修改对应的存储目录，日志目录，工作目录，启动用户等，切记更改完成后要给对应的目录授权
vim /etc/sysconfig/jenkins

JENKINS_HOME="/data/jenkins/work"
JENKINS_USER="operator"
JENKINS_PORT="8080"

#修改启动脚本中的参数，包括war包目录，日志目录，cache目录等
vim /etc/init.d/jenkins

41 JENKINS_WAR="/data/jenkins/jenkins.war"
90 PARAMS="--logfile=/data/jenkins/log/jenkins.log --webroot=/data/jenkins/cache/war --daemon"
107     PARAMS="$PARAMS --accessLoggerClassName=winstone.accesslog.SimpleAccessLogger --simpleAccessLogger.format=combined --simpleAccessLogger.f    ile=/data/jenkins/log/access_log

#启动成功后会输出密码，在日志中也能看到，文件中会存储初始密码：cat /var/lib/jenkins/secrets/initialAdminPassword
#浏览器输入IP+端口，进行访问
~~~



* 问题

  1，打开浏览器，界面一直显示Please wait while Jenkins is getting ready to work ..

  ```text
  需要你进入jenkins的工作目录，打开-----hudson.model.UpdateCenter.xml将 url 中的 
  https://updates.jenkins.io/update-center.json
  更改为https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json
  是国内的清华大学的镜像地址。
  ```

  2，下载插件失败

  进入jenkins系统管理---插件管理---高级---升级站点

  将url修改为清华大学镜像地址：https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json



* 插件

  参数化构建插件：

  Choice Parameter

  Build With Parameters

  Persistent Parameter

  SSH插件：

  Publish over SSH

  SSH plugin

  

