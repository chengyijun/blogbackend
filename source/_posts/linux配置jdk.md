---
title: linux配置jdk
date: 2022-07-15 11:10:21
tags:
---
sudo vim /etc/profile

~~~bash
#Java Env
export JAVA_HOME=/usr/local/jdk
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin
~~~

source /etc/profile
