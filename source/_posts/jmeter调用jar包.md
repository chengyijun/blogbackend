---
title: jmeter调用jar包
date: 2022-07-15 10:43:58
tags:
---
# 1. 导出jar包

> jar包内部逻辑代码如下：（模拟加盐）

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200924094817229.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FiZWxyb3g=,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200924094942484.png#pic_center)

# 2. 将jar包导入jmeter

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200924095255333.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FiZWxyb3g=,size_16,color_FFFFFF,t_70#pic_center)

# 3. 添加 BeanShell取样器

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200924100419144.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FiZWxyb3g=,size_16,color_FFFFFF,t_70#pic_center)

```shell
// 导入jar包
import com.abel.Test;
String param = "rose";
// 调用jar包中的方法
String res = Test.GetMD5(param);
// 将结果赋值给jmeter中的变量进行引用
vars.put("res",res)
```

# 4. 在请求中使用该变量

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200924100618897.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FiZWxyb3g=,size_16,color_FFFFFF,t_70#pic_center)

# 5. 最终jmter就可以调用通过jar包计算出的结果了

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200924100731631.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FiZWxyb3g=,size_16,color_FFFFFF,t_70#pic_center)
