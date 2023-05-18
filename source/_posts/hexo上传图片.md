---
title: hexo上传图片
date: 2022-07-08 10:13:32
tags:
 - hexo
---

1. 开启图片文件夹

~~~bash
# _config.yml
post_asset_folder: true
~~~

2. 安装图片插件

~~~bash
npm install hexo-asset-image --save
~~~

2.1 修改该插件中的`index.js`内容

> `node_modules/hexo-asset-image/index.js` 中大约59行左右内容
~~~javascript

$(this).attr('src', config.root + link + src);
console.info && console.info("update link as:-->" + config.root + link + src);

~~~

3. 创建文章

~~~bash
hexo new "hexo上传图片"
~~~

4. 将图片放到md文件的同名文件夹中

![](hexo上传图片/%E5%9B%BE%E7%89%87%E7%9B%B8%E5%AF%B9%E8%B7%AF%E5%BE%84.jpg)

5. 引用图片

~~~bash
![](hexo上传图片/1.png)
~~~

6. 生成推送看效果

~~~bash
hexo g -d
~~~
