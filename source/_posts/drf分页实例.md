---
title: drf分页实例
date: 2022-07-15 11:19:15
tags:
---
~~~python

from rest_framework import serializers


class StudentSerializers(serializers.ModelSerializer):
    class Meta:
        model = Students  # 设置关联模型     model就是关联模型
        fields = '__all__'  # fields设置字段   __all__表示所有字段
        # fields = ['content', 'location']
        # 排除的字段  直接丢弃  不验证 不入库
        # exclude = ['topic', 'user']
        # exclude = []


class MyPageNumberPagination(PageNumberPagination):
    # 每页默认获取的条数 size=10
    page_size = 10
    # 指定当前获取条数
    page_size_query_param = 'size'
    # 指定当前页数
    page_query_param = "page"


class StudentView(APIView):

    def get(self, request: Request, *args, **kwargs):
        # 获取所有
        students = Students.objects.all()
        # 创建分页对象
        pg = MyPageNumberPagination()
        # 获取分页的数据
        page_roles = pg.paginate_queryset(queryset=students, request=request, view=self)
        # 对数据进行序列化
        ser = StudentSerializers(instance=page_roles, many=True)
        return Response(ser.data)


~~~
