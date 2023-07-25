---
title: django执行原生sql
date: 2023-07-25 19:14:43
tags: django
---
# django执行原生sql

```python
from django.db import connection


def dict_fetchall(cursor):
    columns = [col[0] for col in cursor.description]  # 通过col[0]得到表中列的名字
    return [dict(zip(columns, row)) for row in cursor.fetchall()]


def get_all(sql: str, *args):
    with connection.cursor() as cursor:
        cursor.execute(sql, args)
        dataInfo = dict_fetchall(cursor)  # 调用上面的
    return dataInfo


def get_one(sql: str, *args):
    return get_all(sql, *args)[0]

sql = "select * from api_card where name=%s"
print(get_all(sql, "中行"))
print(get_one(sql, "中行"))
sql2 = "select * from api_card"
print(get_all(sql2))

```

