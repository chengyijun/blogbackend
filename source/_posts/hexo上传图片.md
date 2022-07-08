---
title: hexo上传图片
date: 2022-07-08 10:13:32
tags:
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

2.1 整个替换该插件中的`index.js`内容

~~~javascript
'use strict';
var cheerio = require('cheerio');

// http://stackoverflow.com/questions/14480345/how-to-get-the-nth-occurrence-in-a-string
function getPosition(str, m, i) {
  return str.split(m, i).join(m).length;
}

hexo.extend.filter.register('after_post_render', function (data) {
  var config = hexo.config;
  if (config.post_asset_folder) {
    var link = data.permalink;
    var beginPos = getPosition(link, '/', 3) + 1;
    var appendLink = '';
    // In hexo 3.1.1, the permalink of "about" page is like ".../about/index.html".
    // if not with index.html endpos = link.lastIndexOf('.') + 1 support hexo-abbrlink
    if (/.*\/index\.html$/.test(link)) {
      // when permalink is end with index.html, for example 2019/02/20/xxtitle/index.html
      // image in xxtitle/ will go to xxtitle/index/
      appendLink = 'index/';
      var endPos = link.lastIndexOf('.');
    }
    else {
      var endPos = link.lastIndexOf('/');
    }
    link = link.substring(beginPos, endPos) + '/' + appendLink;

    var toprocess = ['excerpt', 'more', 'content'];
    for (var i = 0; i < toprocess.length; i++) {
      var key = toprocess[i];

      var $ = cheerio.load(data[key], {
        ignoreWhitespace: false,
        xmlMode: false,
        lowerCaseTags: false,
        decodeEntities: false
      });

      $('img').each(function () {
        if ($(this).attr('src')) {
          // For windows style path, we replace '\' to '/'.
          var src = $(this).attr('src').replace('\\', '/');
          if (!(/http[s]*.*|\/\/.*/.test(src)
            || /^\s+\//.test(src)
            || /^\s*\/uploads|images\//.test(src))) {
            // For "about" page, the first part of "src" can't be removed.
            // In addition, to support multi-level local directory.
            var linkArray = link.split('/').filter(function (elem) {
              return elem != '';
            });
            var srcArray = src.split('/').filter(function (elem) {
              return elem != '' && elem != '.';
            });
            if (srcArray.length > 1)
              srcArray.shift();
            src = srcArray.join('/');

            $(this).attr('src', config.root + link + src);
            console.info && console.info("update link as:-->" + config.root + link + src);
          }
        } else {
          console.info && console.info("no src attr, skipped...");
          console.info && console.info($(this));
        }
      });
      data[key] = $.html();
    }
  }
});



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
