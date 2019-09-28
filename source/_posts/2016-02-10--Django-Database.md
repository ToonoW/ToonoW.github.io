---
date: 2016-02-10
layout: post
title: Django Database Setting
category: Python
---

本文主要讲述Django 使用database时需要做的一些设置。



假设我们现在的需求是希望用mysql 代替sqlite3 (django 默认使用sqlite3)，我们首先要修改Django 主应用目录下的`settings.py`，我们要修改里面的`DATABASES`字典。默认是以下内容：

``` python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': 'mydatabase',
    }
}
```

我们可以连接其他的数据库后端，例如mysql，oracle或者postgresql，我们必须提供连接参数。这里是mysql的示例：

``` python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'forusee',
        'USER': 'root',
        'PASSWORD': '123456',
        'HOST': '127.0.0.1',
        'PORT': '3360',
    }
}
```

但是如果是在装了mysql 服务程序的情况下，这里还是缺少mysql 到python 的驱动程序，这里使用的是python3，所以只要`pip install mysqlclient`即可。

然后我们就可以在程序的根目录执行 `python manage.py migrate` 来完成我们的数据库的部署啦。
