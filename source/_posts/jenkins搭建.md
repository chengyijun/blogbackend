---
title: jenkins搭建
date: 2022-07-15 11:20:30
tags:
---
# Jenkins配置教程

## centos7

- 关闭selinux

  ~~~bash
  vi /etc/selinux/config
  ~~~

   将SELINUX=enforcing改为SELINUX=disabled 设置后需要重启才能生效

- 关闭防火墙并禁止自动启动

  ```bash
  systemctl stop firewalld
  systemctl disable firewalld
  systemctl status firewalld 
  ```

- 换源

- 配置静态ip

~~~bash
cd /etc/sysconfig/network-scripts/
~~~

~~~bash
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static
IPADDR=192.168.130.130
NETMASK=255.255.255.0
GATEWAY=192.168.130.1
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
NAME=ens34
DEVICE=ens34
ONBOOT=yes
~~~

## gitlab服务器

~~~bash
# 安装依赖
yum install -y curl policycoreutils-python openssh-server openssh-clients postfix
# 配置sshd
systemctl enable sshd && systemctl start sshd
# 配置邮件服务
systemctl enable postfix && systemctl start postfix
# 下载gitlab
wget https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el6/gitlab-ce-12.4.2-ce.0.el6.x86_64.rpm
# 安装gitlab
rpm -ivh gitlab-ce-12.4.2-ce.0.el6.x86_64.rpm
# 配置gitlab
vi /etc/gitlab/gitlab.rb
external_url 'http://192.168.130.131:82'
nginx['listen_port'] = 82
# 让配置生效
gitlab-ctl reconfigure
gitlab-ctl restart

# gitlab 首次访问会让修改密码(密码要求8位)
root/00000000
~~~

## jenkins服务器

安装jdk

```bash
yum install java-1.8.0-openjdk* -y
```

~~~
2.安装
[root@ticent admin]# yum install java-1.8.0-openjdk.x86_64
3.添加环境变量 vi /etc/profile
JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.282.b08-1.el7_9.x86_64
JRE_HOME=$JAVA_HOME/jre
CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
export JAVA_HOME JRE_HOME CLASS_PATH PATH
4.验证有效性
[root@ticent admin]# java -version
openjdk version "1.8.0_131"
OpenJDK Runtime Environment (build 1.8.0_131-b12)
OpenJDK 64-Bit Server VM (build 25.131-b12, mixed mode)
~~~

下载jenkins

wget ...

上传jenkins

yum install lrzsz -y

rz ...

安装jenkins

rpm -ivh ...

配置jenkis

vim /etc/sysconfig/jenkins

JENKINS_USER="root"

JENKINS_PORT="8888"

启动jenkins

systemctl start jenkins

访问jenkins

<http://192.168.130.132:8888/>

首次访问会要求填写初始密码

cat /var/lib/jenkins/secrets/initialAdminPassword

创建用户如下：

abel/000000

## 测试环境服务器
