---
title: centos7安装python3.8
date: 2022-07-15 10:45:20
tags:
---
# 0. 安装gcc 、c++编译器以及内核文件

~~~bash
yum -y install gcc gcc-c++ kernel-devel
~~~

# 1. 安装最新版openssl

```bash
sudo ./config --prefix=/usr/local/openssl
sudo make && sudo make install
```

安装好之后进行配置
> 备份原来的openssl命令
mv /usr/bin/openssl /usr/bin/openssl.bak
将安装好的bin目录中的openssl文件软连到/usr/bin/openssl
ln -s /usr/local/openssl/bin/openssl /usr/bin/openssl
将安装好的openssl 的openssl目录软连到/usr/include/openssl
ln -s /usr/local/openssl/include/openssl /usr/include/openssl
修改系统自带的openssl库文件，如/usr/local/lib64/libssl.so(根据机器环境而定) 软链到升级后的libssl.so
ln -s /usr/local/openssl/lib/libssl.so.1.1 /usr/local/lib64/libssl.so     ----注意时usr/local下的lib64不是/usr/lib64，两个目录下有同名目录，有可能在/usr/local/lib64/并不存在文件
在/etc/ld.so.conf文件中写入openssl库文件的搜索路径
echo "/usr/local/related/openssl/lib" >> /etc/ld.so.conf
使修改后的/etc/ld.so.conf生效
ldconfig -v
最好查看下openssl版本号，看是否已经更新成最新的，
openssl version -a

# 2. 安装python3.8

```bash
sudo ./configure --prefix=/usr/local/python38 --enable-shared --with-openssl=/usr/local/openssl
sudo make && sudo make install
```

> 安装过程中的报错处理
> 报gcc错误
yum -y install gcc
报zlib错误
yum -y install zlib*
报 '_ctypes'的错误。
yum install libffi-devel -y

# 3. 建立软连接方便使用

```bash
sudo ln -s /usr/local/python38/bin/python3.8 /usr/bin/python38
sudo ln -s /usr/local/python38/bin/pip3.8 /usr/bin/pip38
```

# 4. pip换源 ~/.pip/pip.conf

```bash
[global]
index-url = http://mirrors.aliyun.com/pypi/simple/
[install]
trusted-host = mirrors.aliyun.com
[list]
format=columns
```

# 5. 安装virtualenvwrapper

```bash
sudo pip38 install virtualenvwrapper
```

> 可能会报错 动态库错误  拷贝一下就行了
> error while loading shared libraries: libpython3.8m.so.1.1:cannot open shared object file: No such file or directory

~~~bash
cp /usr/local/python38/lib/libpython3.8.so.1.0 /usr/lib64/
~~~

# 6. 配置virtualenvwrapper

```bash
WORKON_HOME=~/.venvs
VIRTUALENVWRAPPER_VIRTUALENV_ARGS='--no-site-packages'
VIRTUALENVWRAPPER_PYTHON=/usr/local/python38/bin/python3.8
export VIRTUALENVWRAPPER_VIRTUALENV=/usr/local/python38/bin/virtualenv
source /usr/local/python38/bin/virtualenvwrapper.sh
```

在 ~/.bashrc 中写入上面内容
source ~/.bashrc 使其生效

# 7. 创建虚拟环境

```bash
mkvirtualenv py38
```

> 如果报 '--no-site-packages' 无效的不能识别的参数 错误
则需要更新virtualenv

```bash
sudo pip38 install --upgrade virtualenv==16.7.9
```
