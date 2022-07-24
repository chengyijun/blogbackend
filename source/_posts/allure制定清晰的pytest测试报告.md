---
title: allure制定清晰的pytest测试报告
date: 2022-07-15 10:46:13
tags:
---
```python
import allure
import pytest


# feature() 给功能模块取个名字 一个功能模块包含多个功能点 一般一个功能模块就作为一个类
@allure.feature('购物车功能')
class TestShoppingTrolley:
    # story() 给功能点取个名字 一般一个功能点就作为一个方法
    @allure.story('加入购物车')
    def test_add_shopping_trolley(self):
        # 步骤1  因为login() 被装饰器 @allure.step()装饰过 报告中会将步骤详细展示出来
        login('刘春明', '密码')
        # 步骤2
        with allure.step("浏览商品"):
            # attach() 添加文本信息到报告中
            allure.attach('笔记本', '商品1')
            allure.attach('手机', '商品2')
            # attach() 添加截图信息到报告中
            with open('./商品3截图.png', 'rb') as f:
                file = f.read()
            allure.attach(file, '商品3', allure.attachment_type.PNG)
        # 步骤3
        with allure.step("点击商品"):
            pass
        # 步骤4
        with allure.step("校验结果"):
            allure.attach('添加购物车成功', '期望结果')
            allure.attach('添加购物车失败', '实际结果')
            assert 'success' == 'failed'

    @allure.story('修改购物车')
    def test_edit_shopping_trolley(self):
        pass

    @pytest.mark.skipif(reason='本次不执行')
    @allure.story('删除购物车中商品')
    def test_delete_shopping_trolley(self):
        pass


@allure.step('用户登录')
def login(user, pwd):
    print(user, pwd)

```

```bash
pytest cases\test_allure2.py --alluredir ./result/
allure generate ./result -o ./report --clean
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200925164629780.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FiZWxyb3g=,size_16,color_FFFFFF,t_70#pic_center)
