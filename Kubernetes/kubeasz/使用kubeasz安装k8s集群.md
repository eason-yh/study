# 使用kubeasz安装k8s集群

## 服务器初始化

升级内核到4.4 ，内核版本可能存在小版本的不同，以当时的内核版本为准，本次版本为4.4.241

参考文档：https://blog.csdn.net/qq_38773184/article/details/105162366

\1.  #导入elrepo的key，然后安装elrepo的yum源 

\2.  rpm -**import** https://www.elrepo.org/RPM-GPG-KEY-elrepo.org 

\3.  rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm 

\4.  #使用以下命令列出可用的内核相关包 

\5.  yum --disablerepo="*" --enablerepo="elrepo-kernel" list available 

\6.  #长期维护版本lt为4.4,安装lt版本 

\7.  yum --enablerepo=elrepo-kernel install -y kernel-lt 

\8.  #查看内核启动版本 

\9.  cat /boot/grub2/grub.cfg |grep menuentry 

\10. #设置开机内核启动版本，版本以当前安装版本号为准 

\11. grub2-set-default "CentOS Linux (4.4.241-1.el7.elrepo.x86_64) 7 (Core)" 

\12.  

\13. #重启服务器并检查版本 

\14. uname -r 

在规划好的管理节点（manager）上部署ansible

参考安装文档：

https://github.com/easzlab/kubeasz/blob/master/docs/setup/00-planning_and_overall_intro.md

https://github.com/easzlab/kubeasz/blob/master/docs/setup/quickStart.md

\1.  pip install pip --upgrade -i http://mirrors.aliyun.com/pypi/simple/ --trusted-host mirrors.aliyun.com 

\2.  pip install --no-cache-dir ansible -i http://mirrors.aliyun.com/pypi/simple/ --trusted-host mirrors.aliyun.com 

配置免密登录，供ansible使用，本次使用RSA算法

\1.  # 更安全 Ed25519 算法 

\2.  ssh-keygen -t ed25519 -N '' -f ~/.ssh/id_ed25519 

\3.  # 或者传统 RSA 算法 

\4.  ssh-keygen -t rsa -b 2048 -N '' -f ~/.ssh/id_rsa 

\5.   

\6.  ssh-copy-id $IPs #$IPs为所有节点地址包括自身，按照提示输入yes 和root密码

推荐使用 easzup 脚本下载 4.0/4.1/4.2 所需文件；运行成功后，所有文件（kubeasz代码、二进制、离线镜像）均已整理好放入目录/etc/ansible

\1.  # 下载工具脚本easzup，举例使用kubeasz版本2.0.2 

\2.  export release=2.2.1 

\3.  curl -C- -fLO --retry 3 https://github.com/easzlab/kubeasz/releases/download/${release}/easzup 

\4.  chmod +x ./easzup 

\5.  # 指定版本，可以指定使用k8s的哪个版本，使用工具脚本下载

\6.  ./easzup -D –k v1.17.2

必要配置：cd /etc/ansible && cp example/hosts.multi-node ./hosts, 然后实际情况修改此hosts文件

按照hosts中的说明配置相应的地址

 

\# 分步安装

ansible-playbook 01.prepare.yml

ansible-playbook 02.etcd.yml

ansible-playbook 03.docker.yml

ansible-playbook 04.kube-master.yml

ansible-playbook 05.kube-node.yml

ansible-playbook 06.network.yml

ansible-playbook 07.cluster-addon.yml

 

 

 

安装Harobor，使用ansible-playbook 11.harbor.yml进行安装

官方参考文档：https://github.com/easzlab/kubeasz/blob/master/docs/guide/harbor.md

安装之前需要先下载harbor离线安装包到down目录下，本次下载的是1.9.4版本，和安装文件中的对应：https://github.com/goharbor/harbor/releases/tag/v1.9.4

下面会详细列出需要修改的位置

11.harbor.yml中需要修改的为红色框内，只需要harbor是yes，其他的全部是no，因为之前已经安装过了，如果是新的主机并没有安装过相应的软件，请改为yes

![img](file:///C:/Users/ADMINI~1/AppData/Local/Temp/msohtmlclip1/01/clip_image002.jpg)

roles/harbor/defaults/main.yml中可以更改版本号，选择适合自己服务的版本

![img](file:///C:/Users/ADMINI~1/AppData/Local/Temp/msohtmlclip1/01/clip_image004.jpg)

roles/harbor/tasks/main.yml中修改存储路径，按照实际需求路径更改

roles/harbor/templates/harbor-v1.9.yml.j2修改数据库的存储位置，本次对应1.9，所以改1.9的配置文件。请以实际对应版本修改

在hosts中harbor的位置修改为安装到宿主机的IP，内部域名

 

安装完成之后，修改密码步骤

先到harbor的安装目录下，执行停止指令：

COMPOSE_HTTP_TIMEOUT=200 docker-compose down -v

删除安装目录下的database和registry目录

进入harbor子目录，修改harbor.yml里的harbor_admin_password为Potevio123

执行./prepare然后重启服务，COMPOSE_HTTP_TIMEOUT=200 docker-compose up -d

使用docker login来确认是否能成功登录，登录和登出确认方法：

查看root家目录下的 .docker/config.json，存在认证信息证明登录成功

增加新的node节点，docker需要添加证书到/etc/docker下