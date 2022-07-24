---
title: drf的认证-权限节流-版本-控制实例
date: 2022-07-15 11:18:34
tags:
---
~~~python
# Create your views here.
import time

from rest_framework import exceptions
from rest_framework.authentication import BaseAuthentication
from rest_framework.permissions import BasePermission
from rest_framework.request import Request
from rest_framework.response import Response
from rest_framework.throttling import BaseThrottle, SimpleRateThrottle
from rest_framework.versioning import BaseVersioning
from rest_framework.views import APIView

visit_records = {}


class MyBaseVersioning(BaseVersioning):
    """
    版本控制
    """
    versions = ['v1', 'v2']

    def determine_version(self, request: Request, *args, **kwargs):
        version = request.query_params.get('version')
        if version not in self.versions:
            raise exceptions.APIException(detail={'msg': '版本不正确'})
        return version


class MyScopedRateThrottle(SimpleRateThrottle):
    """
    节流控制（继承自drf框架实现）
    """
    scope = 'visit'
    THROTTLE_RATES = {
        'visit': '6/m',  # 每分钟访问6次
    }

    def get_cache_key(self, request, view):
        return request.META.get('REMOTE_ADDR')


class MyThrottle(BaseThrottle):
    """
    节流控制（自定义）
    """

    def allow_request(self, request: Request, view):
        # 拿到访问的IP
        ip = request.META.get('REMOTE_ADDR')
        # 访问记录
        # {ip: [timestamp1, timestamp2]}
        now = time.time()
        if ip not in visit_records.keys():
            # 如果是第一次访问就将ip加入到访问记录中
            visit_records.setdefault(ip, [now])
        # 获取该ip的访问记录
        history = visit_records.get(ip)
        # 如果记录中最先一次的访问时间 超出频率 则移除
        if now - history[-1] > 10.0:
            # 移除过期的访问记录
            history.pop()
        # 判断是否可以访问
        if len(history) < 5:
            # 更新该ip的访问时间记录
            visit_records.get(ip).insert(0, now)
            return True
        return False


class MyPermission(BasePermission):
    """
    权限控制
    """

    def has_permission(self, request, view):
        # 抛出异常或者返回False 表示没有权限
        # raise exceptions.PermissionDenied(detail={'msg': '没有权限哦'})
        # return False
        # 返回True 表示有权限
        return True


class MyAuthentication(BaseAuthentication):
    """
    认证控制
    """

    def authenticate(self, request: Request):
        token = request.query_params.dict().get('token')
        if not token:
            # 第一种结果： 抛出异常，认证失败，并终止向后认证
            raise exceptions.AuthenticationFailed(detail={'msg': '认证失败'})
        if token == '123':
            # 第二种结果： 返回tuple，认证成功，并终止向后认证
            return 'abel', token
        # 第三种结果： 返回None，不进行任何认证，继续向后认证
        return None


class TestView(APIView):
    # 配置认证控制类
    authentication_classes = [MyAuthentication]
    # 配置权限控制类
    permission_classes = [MyPermission]
    # 配置节流控制类
    throttle_classes = [MyScopedRateThrottle]
    # 配置版本控制类  注意此处是单个的类 而不是列表
    versioning_class = MyBaseVersioning

    def get(self, requests, *args, **kwargs):
        print(self)
        print('get')
        print(requests)
        data = dict(list(zip('abcd', range(4))))
        return Response(data)

    def post(self, requests, *args, **kwargs):
        print(self)
        print('post')
        print(requests)
        print(requests.FILES)
        file = requests.FILES.dict().get('file')
        for chuck in file.chunks():
            with open('aaa.png', 'wb') as f:
                f.write(chuck)

        data = dict(list(zip('efgh', range(4))))
        return Response(data)

~~~
