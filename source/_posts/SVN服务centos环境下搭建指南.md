---
title: SVN服务centos环境下搭建指南
date: 2022-07-15 11:22:50
tags:
---
## SVN服务搭建指南

>
>
> 适用范围：centos7系统 + svn服务端 + 自制svn用户密码修改服务（基于DjangoRestFramework开发）

### 1. 创建根目录存储svn仓库和svn配置

~~~bash
 mkdir -p /svn/{svndata,svnpasswd}
~~~

### 2. 启动svn服务

~~~bash
# 启动svn服务
**特别注意**：一定要启动所有仓库的根目录 而不能启动某个具体仓库 否则客户端checkout时会报url路径不存在
# /svn/svndata 是作为所有仓库的根
svnserve -d -r /svn/svndata
        参数：
            -d：表示后台运行守护模式；
            -r：表示svn服务的根目录；

检查svn服务是否启动
ps -ef | grep svnserve
检测svn端口3690是否已经监听：(svn默认启动3690端口)
netstat -lntup | grep 3690
# 如果提示端口已经占用可以先杀死端口,再初始化仓库
pkill svnserve
~~~

### 3. 新建仓库

```bash
# 创建cbct仓库目录
mkdir -p /svn/svndata/cbct
# 初始化仓库
svnadmin create /svn/svndata/cbct
```

### 4. 修改配置文件便于统一管理

```bash
# 拷贝 /svn/svndata/cbct/conf 的两个文件 authz passwd 到 /svn/svnpasswd 目录下 便于管理
# 理由是 权限文件和用户文件默认是分散给各个仓库分散管理的  现在需要集中管理
cp /svn/svndata/cbct/conf/authz /svn/svnpasswd/
cp /svn/svndata/cbct/conf/passwd /svn/svnpasswd/

# 修改 /svn/svndata/cbct/conf 下的 svnserve.conf 文件 指定集中管理的 authz 和 passwd 文件位置
```

```bash
# 需要修改的4个地方 且每个仓库的 svnserve.conf 都需要如此配置
[general]
anon-access = none
auth-access = write
password-db = /svn/svnpasswd/passwd
authz-db = /svn/svnpasswd/authz
```

### 5. 配置用户、组、仓库的关系

```bash
# 创建用户
vim /svn/svnpasswd/passwd
# 按照 用户名 = 密码 的格式追加即可 如下图所示
```

![请添加图片描述](https://img-blog.csdnimg.cn/609cb81f27fb43bdb0dc88a50af5f07d.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5Li_5Li_6ZW_6KGr572p5a2Q6b6Z,size_14,color_FFFFFF,t_70,g_se,x_16)

```bash
# 创建组
```

![请添加图片描述](https://img-blog.csdnimg.cn/aa33463a4aaa43c7990ce537c70137d0.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5Li_5Li_6ZW_6KGr572p5a2Q6b6Z,size_18,color_FFFFFF,t_70,g_se,x_16)

```bash
# 组<-->用户 的关系
# 组名 = 用户名1,用户名2,用户名3
[groups]
group_cbct = chengyijun,xiangyang


```

```bash
# 组<-->仓库 的关系
# [仓库名:/] 表示对 该仓库下的根目录设置权限
# 特别注意：引用组名前面需要带@符号
# 注意仓库下的权限不仅可以绑定组 也可以直接绑定用户
# r表示可读 w表示可写  rw可读写
[cbct:/]
@group_cbct = rw
zhangbin = rw



```

重启svn服务 进行snv访问

```bash
# 停止svn服务
pkill svnserve
# 开启svn服务 切记需要指定所有仓库的根 而不是具体仓库
svnserve -d -r /svn/svndata


# 客户端的svn填入地址为
svn://39.103.186.2/cbct

```

![请添加图片描述](https://img-blog.csdnimg.cn/8485a25a9e3c4335a1ba9f3b6cb7253c.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5Li_5Li_6ZW_6KGr572p5a2Q6b6Z,size_12,color_FFFFFF,t_70,g_se,x_16)

### 6. 配置修改密码的web服务

```bash
# 为了防止修改密码的web服务没有操作 passwd 文件的权限 先进行提权
sudo chmod 777 /svn/svnpasswd/passwd

# 在修改密码的web服务中 指定 passwd 文件所在目录的位置 注意是目录位置 不是文件的全路径
# 进入密码修改服务项目中 指定要操作的svn passwd文件位置
vim /root/svntest/backend/backend/settings.py

```

[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-YdonSDyF-1630997778850)(C:\Users\abel\Desktop\SVN服务搭建.assets\1630993118140.png)]

```bash
# 在djangorestframework项目中启动 修改密码的服务
# 后台启动服务
# nohup 表示不挂起
# -u 参数的作用是取消python stdout的缓存 使其有结果就实时的网mylog文件中写
# 2>&1 表示错误也重定向到 mylog 文件中
# & 表示后台执行
nohup python3 -u manage.py runserver 0.0.0.0:8000 > mylog 2>&1 &

# 查看服务运行情况 看mylog文件
cat mylog
或者 tail -f mylog

# 如果需要停止服务可以进行如下操作
# 查询进程号
ps -ef|grep python3
# 杀死进程
kill 进程号

# 打开浏览器访问就可以修改各个用户的密码
# 旧的密码为创建用户时指定的密码
浏览器访问  http://39.103.186.2:8000/

```

![请添加图片描述](https://img-blog.csdnimg.cn/3854db649c4b4928b40dc0e4be6a6f81.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5Li_5Li_6ZW_6KGr572p5a2Q6b6Z,size_20,color_FFFFFF,t_70,g_se,x_16)
