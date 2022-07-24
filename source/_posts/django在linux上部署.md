---
title: django在linux上部署
date: 2022-07-15 11:09:40
tags:
---
# 目录结构

![image.png](https://img-blog.csdnimg.cn/img_convert/2f277d5945adca259fae6a5c175f08e3.png)

# nginx配置文件 nginx.conf

```bash

#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /usr/local/nginx/conf/mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

 server {
  listen 80;
  server_name localhost;

  root /home/abel/code/website;

  location /static {
   alias /home/abel/code/website/static;
  }
  location / {
   include /usr/local/nginx/conf/uwsgi_params;
   uwsgi_pass 127.0.0.1:8011;
  }

 }
}

```

# uwsgi配置文件  uwsgi.ini   (django)

```bash
[uwsgi]
# 使用nginx反向代理时使用
;socket = 127.0.0.1:8011
# 直接作为web服务器使用 记得host一定要是0.0.0.0
http = 0.0.0.0:8010
# 配置工程目录
chdir = /tmp/pycharm_project_368
# 配置项目的wsgi目录 注意是相对于工程目录 wsgi.py 文件时django框架自动生成的
wsgi-file = testdjango/wsgi.py
# 配置虚拟环境
;home = /root/.venvs/py38
# 配置进程线程的信息
processes = 1
threads = 1
```

# uwsgi配置文件  uwsgi.ini   (flask)

~~~bash

[uwsgi]
#uwsgi启动时，所使用的地址和端口（这个是http协议的）
http = 0.0.0.0:5001
# 虚拟环境路径
virtualenv = /root/.venvs/py38
#指向网站目录
chdir = /tmp/pycharm_project_697
#python 启动程序文件
wsgi-file = myflask.py
#python 程序内用以启动的application 变量名
callable = app
#处理器数
processes = 4
#线程数
threads = 2

~~~

> 注意：flask程序中  app.run(host='0.0.0.0')     host 一定要写 要不然远程访问不了

# uwsgi 相关命令

> 进入虚拟环境 安装uwsgi

~~~bash
pip install uwsgi
~~~

测试uwsgi，创建test.py文件：

~~~python
def application(env, start_response):
    start_response('200 OK', [('Content-Type','text/html')])
    return [b"Hello World"]
~~~

~~~bash
uwsgi --http :8001 --wsgi-file test.py
~~~

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200928084241176.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FiZWxyb3g=,size_16,color_FFFFFF,t_70#pic_center)

- 停止

> `uwsgi --stop /home/abel/code/website/uwsgi.pid`

- 开启 （一定要先进入虚拟环境）

> `uwsgi --ini /home/abel/code/website/uwsgi.ini`

# nginx 相关命令

- 关闭

> `sudo nginx -s quit`

- 重新加载配置文件

> `sudo nginx -s reload`

- 测试配置文件

> `sudo nginx -t -c /home/abel/code/website/nginx.conf`

- 启动

> `sudo nginx -c /home/abel/code/website/nginx.conf`

- 查看服务是否活着

> `ps -ef | grep nginx`
