---
layout: post
title: Python Django（二），数据操作
categories: [Python]
---

Django拥有自带的ORM

<!-- more -->
## 数据库操作
```
# file:settings.py
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'mywebdb', # 数据库名称,需要自己定义
        'USER': 'root',
        'PASSWORD': '123456', # 管理员密码
        'HOST': '127.0.0.1',
        'PORT': 3306,
    }
}
# file:__init__.py
import pymysql
pymysql.version_info = (1, 3, 13, "final", 0)
pymysql.install_as_MySQLdb()

# file:app/models.py
from django.db import models
class user(models.Model):
    # 自增主键
    id = models.AutoField(primary_key = True)
    name =  models.CharField(max_length=10, unique=True)
    # 第一位默认为verbose_name，不写则为属性名
    pwd = models.CharField('密码', max_length=10)
```

## 表关系
### 一对一
```
class User(models.Model):
    id = models.AutoField(primary_key=True)
    name = models.CharField(max_length=10, unique=True)
    # 第一位默认赋值给verbose_name，不写则为属性名
    pwd = models.CharField('密码', max_length=10)


class Card(models.Model):
    uid = models.OneToOneField(User, db_column='uid', primary_key=True, on_delete=models.CASCADE)
    cid = models.CharField('身份证号', max_length=18, unique=True)
```
```sql
-- 关于mysql的级联操作
CREATE TABLE user (
  id int UNSIGNED primary key auto_increment,
  name varchar(10) NOT NULL
  -- 使用InnoDB引擎，默认字符集为utf8，自增的初始值为1
) ENGINE=InnoDB DEFAULT CHARSET=utf8 AUTO_INCREMENT=1;

CREATE TABLE card (
  uid int UNSIGNED primary key,
  cid varchar(10) NOT NULL,
  -- 使用外键约束
  constraint card_uid_fk FOREIGN KEY (uid) REFERENCES user(id) ON DELETE CASCADE ON UPDATE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8 AUTO_INCREMENT=1;
```
### 一对多
```
class Order(models.Model):
    oid = models.CharField('订单号', primary_key=True, max_length=10)
    uid = models.ForeignKey(User, db_column='uid', on_delete=models.CASCADE)
```
### 多对多
```
class Transport(models.Model):
    tid = models.AutoField('交通工具编号', primary_key=True)
    tname = models.CharField('交通工具名称', max_length=10)
    uid = models.ManyToManyField(User)
```

## 查看底层sql
```python
def showsql():
    from django.db import connection
    return connection.queries[-1]['sql']
```

## 聚合函数
MAX()、MIN()、COUNT()、SUM()、AVG()
```
# 返回一个字典，键为赋值的变量名（此里中为m）
User.objects.aggregate(m=Max('id'))
```

## Q、F对象
```
# 多个条件的拼接，& 代表和，| 代表或， ~ 代表非
from django.db.models import Q
User.objects.filter(Q(id=1)|Q(id=2))
# 获取原先值并进行计算或比较，不能用于字符串
from django.db.models import F
User.objects.filter(id=1).update(F=('value')+2)
```

## 原生查询
```
# 查询内容需包含主键，且只能查询
user = User.objects.raw('select * from myApp_user')

# 不必包含主键
from django.db import connection, transaction
with connection.cursor() as cur:
    cur.execute('select name from myApp_user')
    datas = cur.fetchall()
    cur.execute('update myApp_user set name="1" where id=1')
    transaction.commit()
```

## 分页查询
```python
from django.core.paginator import Paginator
def limit_query(request):
    # 获取当前页数
    n = int(request.GET.get('num', 1))
    movie = Movie.object.all()
    # 创建分页器对象，每页20条
    pager = Paginator(movie, 20)
    try:
        perpage_data = pager.page(n)
    except PageNotAnIteger:
        # 默认返回第一页数据
        perpage_data = pager.page(1)
    except EmptyPage:
        # 迭代至最后返回最后一页数据
        perpage_data = pager.page(pager.num_pages)

    return render(request, 'mypage', {'perpage_data':perpage_data})
```

## 上传
```python
# file:models.py
class model(models.Model):
    if request.method == 'POST':
        file = request.FILES.get('file')
        import os
        with open(os.path.join(os.getcwd(), 'upload', file.name), 'wb') as fw:
            fw.write(file.read())
        return HttpResponse('ok')
```
```html
<form action="/test/page1" method="post" enctype="multipart/form-data">
    { % csrf_token %}
    <input type="file" name="file"><br>
    # 未加判空逻辑
    <input type="submit" value="提交">
</form>
```

## cookie和session
```
# file:views.py
# cookie保存在浏览器端，不设超时时间关闭浏览器后自动删除
from django.http import HttpResponse
def cookie_view(request):
    resp = HttpResponse()
    # 设置和修改，path为cookie生效路径
    # max_age按秒计算，expires按时间字符串计算
    resp.set_cookie('cookie名', cookie值, path='/', max_age=超期时间)
    # 获取
    request.COOKIES.get('cookie名', '默认值')
```
```python
def session_view(request):
    request.session['user'] = 'yi'  # 设置session键值对
    request.session.set_expiry()  # 设置有效时间，默认14天
    del request.session['user']  # 删除session中对应数据
    request.session.clear()  # 删除session中所有数据
    request.session.flush()# 清空session中所有数据（包括数据库）
    request.session.session_key  # session_id
    request.session['user']  # 获取对应值
    return HttpResponse('成功')
```