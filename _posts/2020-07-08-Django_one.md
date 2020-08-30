---
layout: post
title: Python Django（一），基础
categories: [Python]
---

一个重量级的 Python Web框架，配备了常用的大部分组件

<!-- more -->
## 基本命令
```
django-admin startproject mywebsite  # 创建项目
python3 manage.py runserver 0.0.0.0:8000  # 运行项目
python3 manage.py startapp  # 创建应用
sudo systemctl start mysqld.service  # 启动数据库
python3 manage.py makemigrations  # 生成或更新迁移脚本
python3 manage.py migrate  # 执行迁移脚本程序
# 如果只想对部分app进行作用则执行如下命令
python3 manage.py makemigrations appname
python3 manage.py migrate appname
python3 manage.py sqlmigrate appname migrations_num  # 查看sql脚本
```

## settings.py
```
INSTALLED_APPS  # 安装的应用
DATABASES  # 数据库配置信息
TEMPLATES  # 模板配置信息
urlpatterns  # 路由配置
STATIC_URL  # 静态资源路由起始位置
STATICFILES_DIRS  # 静态资源查找路径
```

## 应用app
```
# 分布式路由，file:<项目名>/settings.py
INSTALLED_APPS = [
    # ....
    'myapp',  # app的名字
]
# file:<项目名>/urls.py
from django.conf.urls import url
from . import views
urlpatterns = [
    re_path(r'^path/', include('myapp.urls')),
]
# file:myapp/urls.py
urlpatterns = [
    re_path(r'^page', views.page_1),
]
# 此时浏览器的请求地址为127.0.0.1:8000/path/page1
```

## 捕获组传参
```
urlpatterns = [
    re_path(r'^person/(?P<name>\w+)/(?P<age>\d{1,2})', views.myfun),
]
# file:views.py
# 方法的参数必须对应捕获组
def myfun(request, name, age):
    return HttpResponse("姓名:" + name + " 年龄:" + age)
```

## 反向解析url
```
# file:url.py
urlpatterns = [
    re_path(r'^path(\w+)', views.myfun, name="别名"),
]
# file:views.py
def myfun(request):
    return render(request, 'mypage.html')
# mypage.html
<a href="{ % url '别名' '参数' %}"></a>
```

## 静态资源
```
# file: setting.py
STATICFILES_DIRS = [
    os.path.join(BASE_DIR, "static")
]
# file:mypage.html
<img src="/static/images/lena.jpg">
<img src="http://127.0.0.1:8000/static/images/lena.jpg">
{ % load static %}
<img src="{ % static 'images/lena.jpg' %}">
```

## 模板template
```
# 先在工程下创建templates文件夹
TEMPLATES = [
    {
    'BACKEND': 'django.template.backends.django.DjangoTemplates',
    # 'DIRS': [],
    'DIRS': [os.path.join(BASE_DIR, 'templates')], # 添加模板路径
    'APP_DIRS': True, # 是否索引各app里的templates目录
    ...
    },
]
```
```python
# view.py
from django.template import loader
def render_page1(request):
    # 通过loader加载模板
    t = loader.get_template("page1.html")
    # 将t转换成字符串
    html = t.render()
    return HttpResponse(html)

def render_page2(request):
    # 使用 render() 直接加载并响应模板
    from django.shortcuts import render
    # 模板通过dict传参
    return render(requese, 'page2.html',mydict)

mydict = {
 "变量名1":"值1",
 "变量名2":"值2",
}
```
```
# 页面取参
{ { 变量名 }}
# 过滤器
{ {变量|过滤器1:参数值1|过滤器2:参数值2}}
```
```
{ % block block_name %}
# 定义模板块，此模板块可以被子模板重新定义的同名块覆盖
{ % endblock block_name %}
```
```
# 将父模板中所有内容全部继承
{ % extends '父模板名称' %}
{ % block block_name %}
# 子模板块用来覆盖父模板中 block_name 块的内容
{ % endblock block_name %}
```