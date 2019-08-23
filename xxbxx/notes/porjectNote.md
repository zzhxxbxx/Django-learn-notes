@[TOC](第一个Django项目笔记)

## 创建项目
1.通过命令行的方式  
    a>首先要进入安装Django的虚拟环境中     
    b>执行创建命令    
```shell script
django-admin startproject [项目名词]
```
      
2.通过pycharm的方式：
    a> new project->django      
    b> 配置项目路径、python解释器     
    注:
        I.选择已经安装Django的虚拟环境
        II.项目名要符合包命名规范
        
## 运行项目
1 在终端运行：    
    进入项目文件夹中执行命令：   
```shell script
python manage.py runserver
```     
Django默认端口为8000，指定端口号命令：
```shell script
python manage.py runserver
```     
2 在pycharm中运行：
    直接运行就可以，注意：     
    在Edit Configurations中这设置唯一实例，避免重复实例。    
3 访问权限：
    默认设置：仅当前电脑可以访问  
    其它电脑访问需要在`settings.py`下设置`ALLOWED_HOSTS=[]`,允许以什么方式被访问。
    例如：0.0.0.0---允许内网访问

## 项目结构介绍
1. manage.py 以后和项目交互基本上是基于这个文件。
2. settings.py 项目设置，与项目相关的配置。
3. urls.py 配置项目url路由。
4. wsgi.py 项目与WSGI协议兼容的web服务器入口

## 视图函数
1. 视图函数的第一个参数必须是request
2. 视图函数的返回值必须是`django.http.response.HttpResponseBase`的子类对象
