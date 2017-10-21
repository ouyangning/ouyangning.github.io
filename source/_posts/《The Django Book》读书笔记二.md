---
title: 《The Django Book》读书笔记二
date: 2017-10-01 16:32:55
tags: 
- Django
categories: 
- 读书笔记
- 计算机科学
- 软件框架
- The Django Book
---

# 《The Django Book》读书笔记二

## MTV开发模式

把数据存取逻辑、业务逻辑和表现逻辑组合在一起的概念有时被称为软件架构的*Model-View-Controller*(MVC)模式。在这个模式中，Model代表数据存取层，View代表的是系统中选择显示什么和怎么显示的部分，Controller指的是系统中根据用户输入并视需要访问模型，以决定使用哪个视图的那部分。

Django紧紧地遵循这种MVC模式，可以称得上是一种MVC框架。以下是Django中M、V和C各自的含义：

+ M， 数据存取部分，由Django数据库层处理。
+ V， 选择显示哪些数据要显示以及怎样显示的部分，由视图和模板处理。
+ C， 根据用户输入委派视图的部分，由Django框架根据URLconf设置，对给定URL调用适当的Python函数。

由于C由框架自行处理，而Django里更关注的是模型（Model）、模板（Template）和视图（Views），Django也被成为MTV框架。在MTV开发模式中：

+ M代表模型（Model），即数据存取层。该层处理与数据相关的所有事务：如何存取、如何验证有效性、包含哪些行为以及数据之间的关系等，
+ T代表模板（Template），即表现层。该层处理和表现相关的决定：如何在页面或其他类型文档中进行显示。
+ V代表视图（View），即业务逻辑层。该层包含存取模型及调取恰当模板的相关逻辑，你可以把它看作模型与模板之间的桥梁。

## 数据库配置

首先，完成数据库服务器的安装和激活，并在其中建立数据库。然后，在Django的配置文件里，缺省是`settings.py`，打开这个文件并查找数据库配置：

```Python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'mydb',
        'USER': 'root',
        'PASSWORD': 'password',
        'HOST': '127.0.0.1',
        'PORT': '3306'
	}
}
```

之后，如有必要的话，可以在`__init__.py`文件中加上这两行：

```Python
import pymysql
pymysql.install_as_MySQLdb()
```

## 第一个应用程序

接下来创建一个Django APP，一个包含模型，视图和Django代码，并且形式为独立Python包的完整Django应用。

### 关于project和app的区别

+ 一个project包含很多个Django app以及对它们的配置。
+ 技术上,project的作用是提供配置文件，比方说哪里定义数据库连接信息，安装的app列表，`TEMPLATE_DIRS`， 等等。
+ 一个app是一套Django功能的集合，通常包括模型和视图，按Python包结构的方式存在。
+ 例如，Django本身内建有一些app，例如注释系统和自动管理界面。app的一个关键点是它们是很容易移植到其他project和被多个project复用。

对于如何架构Django代码并没有快速成套的规则。如果你只是建造一个简单的Web站点，那么可能你只需要一个app就可以了；但如果是一个包含许多不相关模块的复杂的网站，例如电子商务和社区之类的站点，那么你可能需要把这些模块划分成不同的app，以便日后复用。