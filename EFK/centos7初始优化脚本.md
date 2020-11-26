~~~bash
#!/bin/bash
#author liuhui by 
#this script is only for CentOS 7.x
#check the OS

platform=`uname -i`
if [ $platform != "x86_64" ];then 
echo "this script is only for 64bit Operating System !"
exit 1
fi
echo "the platform is ok"
cat << EOF
+---------------------------------------+
|   your system is CentOS 7 x86_64      |
|      start optimizing.......          |
+---------------------------------------
EOF

#调整时区上海
timedatectl set-local-rtc 1
timedatectl set-timezone Asia/Shanghai

#添加公网DNS地址
cat >> /etc/resolv.conf << EOF
nameserver 114.114.114.114
EOF
#Yum源更换为国内阿里源
yum install wget telnet net-tools bind-utils -y
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.ori
wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo

#添加阿里的epel源
#add the epel
wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
# rpm -ivh http://dl.Fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-8.noarch.rpm

#yum重新建立缓存
yum clean all
yum makecache

#添加内核转发
cat <<EOF > /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

sysctl --system
swapoff -a && sysctl -w vm.swappiness=0
yum install -y yum-utils device-mapper-persistent-data lvm2

#安装vim
yum -y install vim gcc gcc++ make openssl openssl-devel yum-utils device-mapper-persistent-data lvm2 sysstat

#历史命令加时间，在/etc/profile最后加入
export HISTTIMEFORMAT="%Y-%m-%d %H:%M:%S  `whoami` "

#设置最大打开文件描述符数
echo "ulimit -SHn 102400" >> /etc/rc.local
cat >> /etc/security/limits.conf << EOF
* soft nofile 655350
* hard nofile 655350
* soft nproc 655350
* hard nproc 655350
EOF


#禁用selinux
sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config
setenforce 0

#关闭防火墙
systemctl disable firewalld.service 
systemctl stop firewalld.service 

#set ssh
sed -i 's/^GSSAPIAuthentication yes$/GSSAPIAuthentication no/' /etc/ssh/sshd_config
sed -i 's/#UseDNS yes/UseDNS no/' /etc/ssh/sshd_config
systemctl  restart sshd.service


#update soft
yum -y update

cat << EOF
+-------------------------------------------------+
|               optimizer is done                 |
|   it's recommond to restart this server !       |
+-------------------------------------------------+
EOF
~~~

