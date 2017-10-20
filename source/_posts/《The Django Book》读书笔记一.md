---
title: 《The Django Book》读书笔记一
date: 2017-10-01 15:25:01
tags: 
- Django
categories: 
- 读书笔记
---

# 《The Django Book》读书笔记一

## 开始一个项目

项目是Django实例的一系列设置的集合，它包括数据库配置、Django特定选项以及应用程序的特定设置。
转到你创建的目录，运行命令`django-admin.py startproject mysite`这样会在你的当前目录下创建一个目录`mysite`。
`startproject`命令创建一个目录， 包含4个文件：
```
mysite/
	mysite/
		__init__.py
		settings.py
		urls.py
		wsgi.py
	manage.py
```


文件如下：
+ __init__.py : 让Python把该目录当成一个开发包（即一组模块）所需的文件。这是一个空文件，一般你不需要修改它。
+ manage.py ：一种命令行工具，允许你以多种方式与该Django项目进行交互。
+ settings.py ： 该Django项目的设置或配置。
+ urls.py ： Django项目你的URL设置。可视其为你的Django网站的目录。

## 运行开发服务器
Django开发服务是可用在开发期间的，一个内建的，轻量的web服务。
启动服务，运行下面的命令：
`Python manage.py runserver`

报错：

```
You have 13 unapplied migration(s). Your project may not work properly until you apply the migrations for app(s): admin, auth, contenttypes, sessions.
Run 'python manage.py migrate' to apply them.
```

依照指示运行`python manage.py migrate` 即可



默认情况下，`runserver`命令在8000端口启动开发服务器，且仅监听本地连接。要想要更改服务器端口的话，可将端口作为命令行参数传入：
`Python manage.py runserver 8080`
通过指定一个IP地址，可以告诉服务器允许非本地连接访问。如果和其他开发人员共享同一开发站点的话，该功能特别有用：
`python manage.py runserver 0.0.0.0:8000`
完成这些设置后，你本地网络中的其他计算机就可以在浏览器中访问你的IP地址了。