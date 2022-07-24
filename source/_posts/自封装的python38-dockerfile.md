---
title: 自封装的python38-dockerfile
date: 2022-07-15 11:16:57
tags:
---
# 自己封装的py38环境

> Dockfile编写

> 添加启动脚本

~~~bash
sudo vi /var/lib/boot2docker/bootlocal.sh
~~~

~~~bash
# 自动挂在共享文件夹
mkdir -p ~/vmshare/
sudo mount -t vboxsf vmshare ~/vmshare/
#更改时区、时间（/mnt/sda1/localtime是从别的服务器cp过阿里的/usr/share/zoneinfo/Asia/Shanghai）
cp -f /mnt/sda1/localtime /etc/localtime && echo "Asia/Shanghai" > /etc/timezone;
#Docker 中国官方镜像加速
echo "{\"registry-mirrors\": [\"https://registry.docker-cn.com\"]}" > /etc/docker/daemon.json;
~~~

~~~bash
FROM wynemo/python38
RUN cd /root \
    && mkdir .pip \
    && mkdir .venvs \
    # 安装virtualenvwrapper
    && pip install virtualenvwrapper \
    # 修改pip源
    && echo "[global] \n index-url = http://mirrors.aliyun.com/pypi/simple/ \n [install] \n trusted-host = mirrors.aliyun.com \n" > .pip/pip.conf \
    # 修复virtualenv
    && pip install --upgrade virtualenv==16.7.9 \
    # 配置.bashrc文件
    && echo "WORKON_HOME=~/.venvs \n VIRTUALENVWRAPPER_VIRTUALENV_ARGS='--no-site-packages' \n VIRTUALENVWRAPPER_PYTHON=/usr/local/bin/python \n export VIRTUALENVWRAPPER_VIRTUALENV=/usr/local/bin/virtualenv \n source /usr/local/bin/virtualenvwrapper.sh \n" >> .bashrc \
    # 使配置文件生效
    && /bin/bash -c "source .bashrc"
~~~

> 根据Dockfile生成镜像

~~~bash
docker build -t mypy38env:v5 .
~~~

> 删除所有已经关闭的容器

~~~bash
docker rm `docker ps -a|grep Exited|awk '{print $1}'`
~~~
