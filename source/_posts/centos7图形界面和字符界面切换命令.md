---
title: centos7图形界面和字符界面切换命令
date: 2022-07-15 10:48:19
tags:
---
开机以命令模式启动，执行：
systemctl set-default multi-user.target

开机以图形界面启动，执行：
systemctl set-default graphical.target
