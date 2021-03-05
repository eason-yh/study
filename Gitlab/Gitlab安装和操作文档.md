# Gitlab安装和操作文档

### 一，   Gitlab安装

Gitlab有两种安装方式，一种是rpm包安装，一种是yum安装

#### yum安装

添加gitlab国内yum源，并安装gitlab-ce

curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash

 

查看安装包列表和版本

yum list |grep gitlab

安装gitlab-ce

yum install gitlab-ce.x86_64 -y

 

#### rpm包安装

下载对应的安装包

下载地址：https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7

 

使用命令在服务器上下载：

rpm -i https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7/gitlab-ce-13.6.3-ce.0.el7.x86_64.rpm

 

安装gitlab

rpm -i gitlab-ce-13.6.3-ce.0.el7.x86_64.rpm

 

二，   更改配置文件

| 配置文件位置     | 作用               |
| ---------------- | ------------------ |
| /var/opt/gitlab/ | 服务nginx，redis等 |
| /opt/gitlab/     | Gitlab启动命令     |
| /var/log/gitlab  | 日志               |

 

 

#### 常见配置及命令

配置、服务

gitlab配置文件：/etc/gitlab/gitlab.rb

重新加载配置：gitlab-ctl reconfigure

重启服务：gitlab-ctl restart

启动服务：gitlab-ctl start

停止服务：gitlab-ctl stop

 

日志位置

日志路径： /var/log/gitlab

查看所有日志：gitlab-ctl tail

查看nginx日志：gitlab-ctl tail nginx/gitlab_access.log

查看指数据库日志：gitlab-ctl tail postgresql

 

数据库

重启数据库： gitlab-ctl restart postgresql

 

数据库配置文件：（修改内容后，需要修改对应的 /etc/gitlab/gitlab.rb 配置，否则重新加载gitlab配置文件后修改会失效）

 

/var/opt/gitlab/gitlab-rails/etc/database.yml

/var/opt/gitlab/postgresql/data/postgresql.conf

/var/opt/gitlab/postgresql/data/pg_hba.conf

 

#### 测试Gitlab邮件发送

更改配置文件

gitlab_rails['smtp_enable'] = true
 gitlab_rails['smtp_address'] = "smtp.sina.com"
 gitlab_rails['smtp_port'] = 465
 gitlab_rails['smtp_user_name'] = "*****@sina.com"
 gitlab_rails['smtp_password'] = "*****"
 gitlab_rails['smtp_domain'] = "sina.com"
 gitlab_rails['smtp_authentication'] = "login"
 gitlab_rails['smtp_enable_starttls_auto'] = true
 gitlab_rails['smtp_tls'] = true
 gitlab_rails['gitlab_email_from'] = "****@sina.com"
 user['git_user_email'] = "*****@sina.com"

 

更新gitlab配置文件

gitlab-ctl reconfigure

重启gitlab服务

gitlab-ctl restart

 

进入gitlab交互端进行测试

gitlab-rails console

输入发送邮件命令

Notify.test_email('xxx@xxx.com','email title','email content desc').deliver_now

 

确定邮件发送成功
