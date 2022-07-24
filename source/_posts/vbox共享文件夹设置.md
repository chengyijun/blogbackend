---
title: vbox共享文件夹设置
date: 2022-07-15 11:15:19
tags:
---
# vbox共享文件夹设置

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201209181839978.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FiZWxyb3g=,size_16,color_FFFFFF,t_70)

~~~bash
sudo mount -t vboxsf vmshare ~/vmshare/ 
~~~

> *注意*   vboxsf 是vbox文件格式
>
> vmshare是前面设置的共享文件夹的名称
