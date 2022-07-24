---
title: python+appium测试移动端app流程
date: 2022-07-15 11:13:49
tags:
---
### 1. 下载android sdk

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201207152502681.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FiZWxyb3g=,size_16,color_FFFFFF,t_70)

### 2. 配置 android sdk

1. 配置 ANDROID_HOME和Path

~~~bash

ANDROID_HOME
D:\Program Files (x86)\Android\android-sdk

Path
%ANDROID_HOME%
%ANDROID_HOME%\tools
%ANDROID_HOME%\platform-tools
%ANDROID_HOME%\build-tools

~~~

2. 配置Path

   ​    ![img](https://img-blog.csdnimg.cn/20190308171511267.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwMjc5OTY0,size_16,color_FFFFFF,t_70)

### 3. 安装模拟器

- 自行下载安装android模拟器即可 （此处安装的是 夜神模拟器）
- *注意* 为了版本一致性 防止出错 可以将android sdk根目录\platform-tools\adb.exe 复制一份出来并改名nox_adb.exe 覆盖到夜神模拟器根目录
- ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201207155821625.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201207152554623.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FiZWxyb3g=,size_16,color_FFFFFF,t_70)

### 4. 通过adb连接模拟器

1. 通过adb查询一下连接的设备列表

 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201207152628161.png)

2. 手动连接一下模拟器 *注意* 不同的模拟器 端口可能不一样 夜神模拟器是62001 其他的可以查询

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201207152634230.png)

### 5. 配置appium

1. 四项基本配置

   ~~~json
   {
     "deviceName": "127.0.0.1:62001",
     "platformName": "Android",
    "appPackage": "com.bignox.google.installer",
     "appActivity": ".MainActivity"
   }
   ~~~

   - 上述第3项 第4项 可以通过如下指令查询到  *前提*是在模拟器上将待测软件打开到启动界面
   > 以下命令可以获取当前启动状态的app的 包名和activity名

   ~~~bash
   adb shell
   dumpsys activity | grep "mFoc"
   ~~~

2. 配置好四项基本配置之后，可以开启会话 appium就得到了模拟器的图像

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201207152835363.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FiZWxyb3g=,size_16,color_FFFFFF,t_70)

### 6. 安装 appium-python-client

> 此模块用来安装appium 像selenium一样通过python来操作

~~~bash
pip install appium-python-client
~~~

### 7. 通过元素id进行定位

- 可以借助android sdk目录下的

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201207152805176.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FiZWxyb3g=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201207152812418.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FiZWxyb3g=,size_16,color_FFFFFF,t_70)

### 8. python代码示例

~~~python
# -*- coding:utf-8 -*-
"""
@author: chengyijun
@contact: cyjmmy@foxmail.com
@file: test1.py
@time: 2020/12/7 15:06
@desc:
"""


def main():
    from appium import webdriver

    caps = {}
    # 平台名称 Android or Ios
    caps["platformName"] = "Android"
    # 包名
    caps["appPackage"] = "com.bignox.google.installer"
    # activity名
    caps["appActivity"] = ".MainActivity"
    # 键盘输入设置 （不是必须的）
    caps["resetKeyboard"] = True
    caps["unicodeKeyboard"] = True
    # 连接appium服务器 路径'/wd/hub'是固定的
    driver = webdriver.Remote("http://localhost:4723/wd/hub", caps)
    # 通过元素id定位 并操作点击
    driver.find_element_by_id('com.bignox.google.installer:id/install').click()
    # 退出driver
    driver.quit()


if __name__ == '__main__':
    main()

~~~
