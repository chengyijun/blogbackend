---
title: 使用drf框架编写api
date: 2022-07-15 11:11:12
tags:
---
# 安装djangorestframework

~~~py
pip install djangorestframework
~~~

# 修改常用配置

~~~python
DEBUG = True

ALLOWED_HOSTS = ['*']

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'api',
        'USER': 'abel',
        'PASSWORD': 'abel',
        'HOST': '127.0.0.1',
        'POST': 3306,

    }
}

INSTALLED_APPS = [
    ...
    'rest_framework',
    '你的app名称',
]

LANGUAGE_CODE = 'zh-hans'
TIME_ZONE = 'Asia/Shanghai'
USE_TZ = False
~~~

项目的同名目录下的 ~__init__.py~

~~~python

import pymysql

pymysql.install_as_MySQLdb()
~~~

# url 路由分发

~~~python
from django.urls import path, include

urlpatterns = [
    path('api/', include("api.urls", namespace='api'))
]
~~~

~~~python
from django.urls import path, re_path

from api import views

app_name = 'api'
urlpatterns = [
    path('login/', views.LoginView.as_view(), name='login'),
    path('code/', views.CodeView.as_view(), name='code'),
    path('tempauth/', views.TempAuthView.as_view(), name='tempauth'),

    # APIView
    path('articles/', views.ArticleView.as_view(), name='articles'),
    re_path(r'article/(?P<id>\d+)/$', views.ArticleDetailView.as_view(), name='article-detail'),
    # 等价于上面的写法 建议使用上面一种 便于理解
    # path('article/<id>/', views.ArticleDetailView.as_view(), name='article-detail'),
]
~~~

# view 编写

- 为了方便复用，尽量使用 ListAPIView, CreateAPIView, （此二者可以合并为 ListCreateAPIView）

- RetrieveAPIView, UpdateAPIView, DestroyAPIView

- 按照上面编写的路由规则  访问url形式为：

- ```
  http://127.0.0.1:8000/api/articles/
  http://127.0.0.1:8000/api/article/78/ 
  ```

```python
class ArticleView(CreateAPIView, ListAPIView):
    # 查询结果集
    queryset = Articles.objects.all()
    # 序列化器类
    # serializer_class = ArticleSerializer
    # 自定义分页器
    pagination_class = MyPageNumberPagination
    # 自定义queryset结果集
    filter_backends = [ArticleFilter]

    def get_serializer_class(self):
        # 根据不同请求 加载不同的序列化器
        if self.request.method == 'POST':
            return ArticleSerializer
        elif self.request.method == 'GET':
            return ArticleSerializerForList

    # 钩子 重写该方法可以在 序列化完成 数据入库之前的时机  插入其他字段值
    def perform_create(self, serializer):
        print(serializer.initial_data)
        userinfo_id = serializer.initial_data.get('user')
        topic_id = serializer.initial_data.get('topic')
        # serializer.save() 方法的内部 又调用了 序列化器中的 create()方法
        article_obj = serializer.save(topic_id=topic_id, user_id=userinfo_id)
        return article_obj

```

```python
class ArticleDetailView(RetrieveAPIView, UpdateAPIView, DestroyAPIView):
    # 查询结果集
    queryset = Articles.objects.all()
    # 序列化器类
    serializer_class = ArticleSerializerForList
    # django 默认的主键形参为pk 如果改用其他 应通过 lookup_field 显试指定出来
    lookup_field = 'id'
```

分页器

```python
from rest_framework.pagination import LimitOffsetPagination


class MyPageNumberPagination(LimitOffsetPagination):
    # 默认数据量
    default_limit = 10
    # 最大数据量
    max_limit = 50
    # 数据量
    limit_query_param = 'limit'
    # 起始偏移位置
    offset_query_param = 'offset'

    def get_paginated_response(self, data):
        """
        overwrite 不要父类给的数据格式
        :param data:
        :return:
        """
        return Response(data)

    def get_offset(self, request):
        """
        overwrite 让起始偏移量 始终为0
        :param request:
        :return:
        """
        return 0
```

结果集过滤器

```python
class ArticleFilter(BaseFilterBackend):

    def filter_queryset(self, request, queryset, view):
        min_id = request.query_params.get('min_id')
        max_id = request.query_params.get('max_id')
        if min_id:
            # 处理上滑 翻页
            queryset = queryset.filter(id__lt=min_id).order_by('-id')
        elif max_id:
            # 处理 下拉刷新
            queryset = queryset.filter(id__gt=max_id).order_by('-id')
        else:
            queryset = queryset.order_by('-id')
        return queryset
```

序列化器

```python
class ArticleImageSerializer(serializers.ModelSerializer):
    class Meta:
        model = ArticleImages  # 设置关联模型     model就是关联模型
        # fields = '__all__'  # fields设置字段   __all__表示所有字段
        exclude = []
```

```python
class ArticleSerializer(serializers.ModelSerializer):
    # 此处为序列化器的嵌套  应用于关联表
    images = ArticleImageSerializer(many=True)

    class Meta:
        model = Articles  # 设置关联模型     model就是关联模型
        # fields = '__all__'  # fields设置字段   __all__表示所有字段
        # fields = ['content', 'location']
        # 排除的字段  直接丢弃  不验证 不入库
        exclude = ['topic', 'user']
        # exclude = []

    # view中数据保存方法 save()执行之前  会调用该方法 通常应用在 序列化器嵌套情况下 一并将嵌套表的字段 入库
    def create(self, validated_data):
        images = validated_data.pop('images')
        article_obj = Articles.objects.create(**validated_data)
        images_obj = [ArticleImages.objects.create(**image, article=article_obj) for image in images]
        article_obj.images = images_obj
        return article_obj

    # 验证字段 钩子
    # def validate_content(self, value):
    #     """ 验证是否还正在拍卖"""
    #     print('正在验证', value)
    #     # item_id = self.initial_data.get('item')
    #     # exists = models.AuctionItem.objects.filter(id=item_id, status=3).exists()
    #     # if not exists:
    #     #     raise exceptions.ValidationError('拍卖商品不存在或已成交')
    #     return value
```

```python
class ArticleSerializerForList(serializers.ModelSerializer):
    class Meta:
        model = Articles
        exclude = []
 # 定义模型中没有的数据字段
    topic = serializers.SerializerMethodField(label='所属话题', default='默认话题内容...', read_only=True)
    user = serializers.SerializerMethodField(label='所属用户手机号', default='13333333333', read_only=True)
    images = serializers.SerializerMethodField(label='文章包含的图片', default='', read_only=True)

    # 钩子方法   固定格式  get_xx(self,value)  xx表示上面对应的自定义字段 value表示当前的model对象
    def get_images(self, value):
        images = ArticleImages.objects.filter(article_id=value.id).all()
        datas = []
        for image in images:
            datas.append({
                'cos_path': image.cos_path,
                'key': image.key
            })
        return datas

    def get_topic(self, value):
        topic = value.topic
        return model_to_dict(instance=topic)

    def get_user(self, value):
        user = value.user
        return model_to_dict(instance=user)
```

# model中外键和多对多关系的用法

> related_name 用作反向关联取得别名 如果表A中有两个外键字段 指向表B中的id  表B方向关联查找数据的时候就不知道应该反向关联谁了 related_name 显示指定别名可以防止此类情况
> 关联自己  可以使用self

~~~python
class Comments(models.Model):
    content = models.CharField(verbose_name='评论内容', max_length=255)
    depth = models.IntegerField(verbose_name='评论级别')

    # 外键
    article = models.ForeignKey(verbose_name='所属文章', to='Articles', on_delete=models.CASCADE, null=True, blank=True)
    user = models.ForeignKey(verbose_name='所属用户', to='Users', on_delete=models.CASCADE, null=True, blank=True)
    root = models.ForeignKey(verbose_name='所属根评论', to='self', related_name='roots', null=True, blank=True,
                             on_delete=models.CASCADE)
    reply = models.ForeignKey(verbose_name='上级评论', to='self', related_name='replys', null=True, blank=True,
                              on_delete=models.CASCADE)
~~~

~~~python
class Articles(models.Model):
    content = models.CharField(verbose_name='文章内容', max_length=255)
    location = models.CharField(verbose_name='地理位置', max_length=255)
    view_count = models.PositiveIntegerField(verbose_name='浏览数', default=0)
    like_count = models.PositiveIntegerField(verbose_name='点赞数', default=0)
    comment_count = models.PositiveIntegerField(verbose_name='评论数', default=0)

    # 外键
    topic = models.ForeignKey(verbose_name='所属话题', to='Topics', on_delete=models.CASCADE, null=True, blank=True)
    user = models.ForeignKey(verbose_name='所属用户', to='Users', on_delete=models.CASCADE, null=True, blank=True)
    # 浏览记录 多对多
    viwers = models.ManyToManyField('Users', related_name='views')
~~~

> 上面的多对多关系  起始就是建立了中间表 中间表使用两个外键关联两张表 构成多对多关系  与手动创建中间表效果一样

~~~python
class ViewRecords(models.Model):
     # 外键
     user = models.ForeignKey(verbose_name='所属用户', to='Users', on_delete=models.CASCADE, null=True, blank=True)
     article = models.ForeignKey(verbose_name='所属文章', to='Articles', on_delete=models.CASCADE, null=True, blank=True)
~~~

# 跨表取值

- A表含有主键指向B表 现有A表的模型对象a 可以直接通过 __a.主键名__获取B表模型对象从而得到B表的数据 （主动查询）
- 反之，现有B表的模型对象b，想通过b取到A表的值，可以通过  __b.a表小写表名_set__ 来得到A表中的数据   _set 为固定格式写法

# 数据库填充数据脚本

> 原理 就是在外部通过加载django环境 让脚本中可以利用django的model快速操作数据库

~~~python
import os
import random
import sys
import uuid

sys.path.append(os.getcwd())
os.environ.setdefault("DJANGO_SETTINGS_MODULE", "miniprogramapi.settings")
import django

django.setup()

from api.models import Topics, Articles, Users, ArticleImages

# 新增用户
for _ in range(20):
    r = random.randint(111111111, 999999999)
    Users.objects.create(
        phone=f'13{r}',
        token=uuid.uuid4(),
        nickname=f'哨兵{r}号',
        avatar='http://himg.bdimg.com/sys/portrait/item/c9eee4b89ce69da5e4b89ce5be80e79a84e78caaa629.jpg'
    )

# 新增话题
for _ in range(20):
    r = random.randint(1, 200)
    Topics.objects.create(
        content=f"我是话题{r}",
        hot=r
    )

# 新增文章
for _ in range(20):
    r = random.randint(1, 200)
    Articles.objects.create(
        content=f"文章内容{r}",
        location=f"明珠路{r}",
        view_count=1,
        like_count=1,
        comment_count=1,
        topic=Topics.objects.get(id=random.choice([topic.id for topic in Topics.objects.all()])),
        user=Users.objects.get(id=random.choice([user.id for user in Users.objects.all()]))

    )
~~~

# 序列化器中数据的校验

1. 方法一 指定数据行验证类  利用 __call__()方法进行校验

~~~

class LoginSerializer(serializers.Serializer):
    phone = serializers.CharField(label='手机号', validators=[PasswordValidator()])
    


class PasswordValidator(object):
    def __init__(self):
        pass

    def __call__(self, value, *args, **kwargs):
        if len(value) != 11:
            raise serializers.ValidationError('手机号码长度不正确')
        if not re.match(r'^\d+$', value):
            raise serializers.ValidationError('手机号必须为数字')

    def set_context(self, serializer_field):
        # print(serializer_field)
        pass
~~~

2. 方法二 指定自定义验证函数

~~~python
class LoginSerializer(serializers.Serializer):
    phone = serializers.CharField(label='手机号', validators=[foo])


# 自定义的验证函数 value表示被验证字段
def foo(value):
    pass
~~~

3. 方法三 使用钩子

~~~python
class LoginSerializer(serializers.Serializer):
    phone = serializers.CharField(label='手机号')
    
    
    # 以 validate_xxx  形式开头的方法是验证钩子  会自动调用进行字段验证
    def validate_phone(self, value):
         print(value + '``````````')
         raise serializers.ValidationError('aaaa')
         return value

~~~

# 登录认证

~~~python

REST_FRAMEWORK = {
    # 未认证情况下 默认用户和令牌为None
    'UNAUTHENTICATED_TOKEN': None,
    'UNAUTHENTICATED_USER': None,
    # 可以通过全局指定的方式 指定登录认证的中间件位置
    'DEFAULT_AUTHENTICATION_CLASSES': ['api.utils.LoginAuthentication']
}

~~~

~~~python

from rest_framework.authentication import BaseAuthentication

from api.models import Users


class MyGeneralAuthentication(BaseAuthentication):
    def authenticate(self, request):
        """
        :param request:
        :return: None -表示本认证器不做任何处理直接交给下一个认证器 认证成功会返回一个元组  (user,token) 表示认证成功 并分贝绑定在
        request.user request.auth 属性上
        """
        # 获取token
        # 注意： 特别值得注意的是  前端 header = {
        #   'authentication': 578fff92-e481-4223-a49a-427cdadfd915
        # }
        # drf框架中接收的时候   需要 加 HTTP_ 前缀 并将key全大写
        token = request.META.get('HTTP_AUTHENTICATION', None)
        # 如果token没有带过来
        if not token:
            return None
        # 如果token是错误的
        user = Users.objects.filter(token=token).first()
        if not user:
            return None
        # 通过认证  会返回一个元组（登录用户对象，登录用户token） 并将其作为request属性绑定上  使用的时候可以通过
        # request.user request.auth 拿到
        # print(user, token)
        return user, token


~~~

之后可以在view中指定 认证器类名 处理认证过程的业务逻辑可以通过 重写方法来进行

~~~~python

class ArticleDetailView(RetrieveAPIView, UpdateAPIView, DestroyAPIView):
    # 查询结果集
    queryset = Articles.objects.all()
    # 序列化器类
    serializer_class = ArticleSerializerForList
    # django 默认的主键形参为pk 如果改用其他 应通过 lookup_field 显试指定出来
    lookup_field = 'id'
    # 指定登录认证器
    authentication_classes = [MyGeneralAuthentication]

    def get(self, request, *args, **kwargs):
        response = super().get(request, *args, **kwargs)

        # 此处验证是否登录  如果登录了 就可以更新浏览记录
        if not request.auth:
            return response

        # 检查是否浏览记录已经存在 不用再添加了
        # 拿到该文章所有的浏览记录
        id = kwargs.get('id')
        article = Articles.objects.filter(id=id).first()
        user = request.user

        if user in [u for u in article.viwers.all()]:
            return response

        # 向中间表添加数据  也就是浏览记录
        article.viwers.add(user)

        return response

~~~~

# celery的使用

~~~python
pip install celery
pip install redis
pip install eventlet # eventlet 只有windows下才需要安装 linux不需要安装
~~~

~~~python

from celery import Celery
"""
broker 指定任务队列
backend 指定结果队列
"""
app = Celery('tasks',
             backend='redis://localhost:6379/0',
             broker='redis://localhost:6379/0')

"""
使用 @app.task 装饰后就成为了一个 可以被celery调度的任务
"""
@app.task
def add(x, y):
    return x + y
~~~

~~~python
from test import add
import time
#不要直接 add(4, 4)，这里需要用 celery 提供的接口 delay 进行调用
result = add.delay(4, 4)  
print(result.id)

# 可以通过 reslut.ready() 判断add这个task是否处理结束
while not result.ready():
    time.sleep(1)
# 可以通过 result.get() 来获取task的结果
print(f'task done: {result.get()}')

~~~

> 注意：worker 的启动命令
> celery worker -A taskproj -l info -P eventlet
> 只有windows平台需要指定  -P eventlet 在linux下不需要

# celery 与 django 嵌入使用

## django 项目名为： miniprogramapi  项目同名目录  miniprogramapi  创建的app名称为 testcelery

1. 在项目的同名目录下创建celery.py文件   miniprogramapi/miniprogramapi/celery.py

   > 文件名必须为 celery.py 固定不能变

```python
import os
from celery import Celery

# 设置django环境
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'miniprogramapi.settings')

app = Celery('testcelery')
# 使用CELERY_ 作为前缀，在settings中写配置
app.config_from_object('django.conf:settings', namespace='CELERY')
# 发现任务文件每个app下的task.py
app.autodiscover_tasks()

```

2. 在项目的同名目录下 `__init__.py` 文件中添加内容

   ```python
   from .celery import app as celery_app
   
   __all__ = ['celery_app']
   ```

3. 在项目同名文件夹 settings.py 中添加关于 celery的配置

   ```python
   # ******************************* celery的配置 ***************************
   # Broker配置，使用Redis作为消息中间件
   CELERY_BROKER_URL = 'redis://127.0.0.1:6379/0'
   # BACKEND配置，这里使用redis
   CELERY_RESULT_BACKEND = 'redis://127.0.0.1:6379/0'
   # 结果序列化方案
   CELERY_RESULT_SERIALIZER = 'json'
   ```

4. 在创建的app中定义 tasks.py 文件

   > 文件名必须叫 tasks.py

   ```python
   from celery import shared_task
   
   
   @shared_task
   def add(x, y):
       return x + y
   
   
   @shared_task
   def mul(x, y):
       return x * y
   ```

5. 启动 worker的命令

   ```
   celery worker -A miniprogramapi -l info -P eventlet
   ```

6. 编写urls

   ```python
   from django.urls import path
   
   from testcelery.views import TestCelery, ResultCelery
   
   app_name = 'testcelery'
   urlpatterns = [
       # 执行触发celery task
       path('celery/', TestCelery.as_view(), name='celery'),
       # 查询celery task的执行结果
       path('result/', ResultCelery.as_view(), name='result'),
   
   ]
   
   ```

7. 编写视图

   ```python
   # Create your views here.
   from celery.result import AsyncResult
   from rest_framework.response import Response
   from rest_framework.views import APIView
   
   from miniprogramapi import celery_app
   from testcelery import tasks
   
   
   class TestCelery(APIView):
       def get(self, request, *args, **kwargs):
           res = tasks.add.delay(1, 3)
           # 任务逻辑
           data = {'status': 'successful', 'task_id': res.task_id}
           return Response(data)
   
   
   class ResultCelery(APIView):
       def get(self, request, *args, **kwargs):
           result_id = request.query_params.dict().get('result_id')
           # result_id = '1f9b12da-f4da-4562-a6d1-ad2d7e18d86b'
           result_obj = AsyncResult(id=result_id, app=celery_app)
           data = {
               'status': 1,
               'state': result_obj.state,
               'result': result_obj.get()
           }
           return Response(data)
   
   ```

### 扩展

　　除了redis、rabbitmq能做结果存储外，还可以使用Django的orm作为结果存储，当然需要安装依赖插件，这样的好处在于我们可以直接通过django的数据查看到任务状态，同时为可以制定更多的操作，下面介绍如何使用orm作为结果存储。

1. 安装

```python
pip install django-celery-results
```

2. 配置settings.py，注册app

```python
INSTALLED_APPS = (
    ...,
    'django_celery_results',
)
```

3. 修改backend配置，将redis改为django-db

```python
# CELERY_RESULT_BACKEND = 'redis://127.0.0.1:6379/0' # BACKEND配置，这里使用redis

CELERY_RESULT_BACKEND = 'django-db'  #使用django orm 作为结果存储
```

4. 修改数据库

```python
python3 manage.py migrate django_celery_results
```
