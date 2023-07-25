---
title: django数据数据生成脚本
date: 2023-07-25 19:13:46
tags: django
---
# django数据数据生成脚本

```python
# 注意： db_init.py 此文件需要放在与manage.py同级目录
import os
import sys
import django
from pathlib import Path

# 注意： 修改为你的项目名称
project_name = "djninja"

base_dir = Path(__file__).absolute().parent
sys.path.append(base_dir)

os.environ.setdefault('DJANGO_SETTINGS_MODULE', f'{project_name}.settings')
django.setup()
# ------------------------------------------------
# 注意：你自己的models一定要在 django.setup() 之后导入 否则会报错
from api.models import Person, Card

person = Person.objects.create(name="jojo")
card1 = Card.objects.create(name="中行", person=person)
```

