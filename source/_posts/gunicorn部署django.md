---
title: gunicorn部署django
date: 2023-07-25 17:22:38
tags:
 - gunicorn
 - django
---
# gunicorn部署django



#### 1 收集静态文件

```python
# settings.py
STATIC_URL = '/static/'
# 项目中静态文件的位置
STATICFILES_DIRS = [
    BASE_DIR.joinpath('app/static'),
]
# 通过 python manage.py collectstatic 命令收集上面的静态文件放置的位置
STATIC_ROOT = BASE_DIR.joinpath('collect_static')
```

```bash
python manage.py collectstatic
```



#### 2 测试阶段

```python
DEBUG = True
```

```python
from django.contrib import admin
from django.urls import path,re_path
from django.contrib.staticfiles.urls import staticfiles_urlpatterns

urlpatterns = [
    path('admin/', admin.site.urls),
]
# 以下内容可以保证 debug模式下正常访问到静态资源
urlpatterns += staticfiles_urlpatterns()
```



#### 3 部署阶段

```python
DEBUG = False
ALLOWED_HOSTS = ["*"]
```

```python
from django.contrib import admin
from django.urls import path,re_path
from django.views import static
from django.conf import settings
urlpatterns = [
    path('admin/', admin.site.urls),
    # 以下内容可以保证 非debug模式下正常访问到静态资源
    re_path(r'^static/(?P<path>.*)$', static.serve, {'document_root': settings.STATIC_ROOT}, name='static')
]
```



```bash
# 测试启动
gunicorn 项目名.wsgi -b 0.0.0.0:9999

# 正式启动 通过加载配置文件 配置文件在下方
gunicorn -c gunicorn.config.py 项目名.wsgi:application
```



```python
# gunicorn.config.py
import logging
import logging.handlers
from logging.handlers import WatchedFileHandler
import os
import multiprocessing

bind = '0.0.0.0:9999'      # 绑定ip和端口号
# chdir = '/opt/workspace/bookstore'  # 目录切换
# backlog = 500              # 监听队列
timeout = 60                 # 超时
worker_class = 'gevent' # 使用gevent模式，还可以使用sync 模式，默认的是sync模式
workers = multiprocessing.cpu_count() * 2 + 1    # 进程数
threads = 2  # 指定每个进程开启的线程数
loglevel = 'info'  # 日志级别，这个日志级别指的是错误日志的级别，而访问日志的级别无法设置
access_log_format = '%(t)s %(p)s %(h)s "%(r)s" %(s)s %(L)s %(b)s %(f)s" "%(a)s"'
accesslog = "log/gunicorn_access.log"  # 访问日志文件
errorlog = "log/gunicorn_error.log"    # 错误日志文件
```

