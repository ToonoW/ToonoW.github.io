---
title: 踩过才知道的Django坑
date: 2017-08-26 01:23:30
tags:
- Python
- Django
categories:
- Python
- Web
---

## Model的字段null和blank设置不为空，但是看数据库怎么还是存了空白值？

在新建一个Model对象的时候发现字段不完整也能够save，但是我设计Model类的时候已经把一些必填的字段设置了`null=True`和`blank=True`，按理说存储的值是不允许为空的。还以为是Sqlite3的问题，换到了MySQL也是没有变化。所以就找到关于`null`和`blank`相关的文章，发现了Django没有按照我们预期进行的原因。

> Django规定储存空字符串来代表空值, 当从数据库中读取NULL或空值时都为空字符串。

假设我们的Model A，有CharField字段`null=False`和`black=False`a。我实验发现，`item = A()`之后，字段a已经存在了空字符串值`''`。也就是在新对象创建之后，就算不给a字段赋值，这个对象也是可以正常调用`save`方法的，其a的值就是空字符串，类型为`<class 'str'>`。

同时我也尝试给a字段赋值`None`，发现并不能够正常调用`save`方法，提示此字段不允许为`NULL`。我然后去修改了A模型，给它增加一个`null=True`和`black=True`的字段b，`item = A()`之后发现字段b的值为`None`，类型为`<class 'NoneType'>`。然后多次一举地给它赋值`None`，然后顺利地调用了`save`方法。然后查看数据库是`NULL`，通过Django的ORM读出也是None。但是这就和我看到的文档不一样了，以下引用文档。

> `Field.null`[¶](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.Field.null)
>
> ​       
>
> If `True`, Django will store empty values as `NULL` in the database. Default is `False`.
>
> Avoid using [`null`](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.Field.null) on string-based fields such as [`CharField`](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.CharField) and [`TextField`](https://docs.djangoproject.com/en/1.11/ref/models/fields/#django.db.models.TextField). If a string-based field has`null=True`, that means it has two possible values for “no data”: `NULL`, and the empty string. In most cases, it’s redundant to have two possible values for “no data;” the Django convention is to use the empty string, not `NULL`. 

按照上面的文档说的，我们通过ORM取出数据，b字段应该是空字符串，但是我取到的是`None`就让我疑惑了。我在Django的邮件列表上留言，希望能获得解答。

所以如果希望某个字段必须被赋值，那么就需要我们手动检查这个字段是否被赋值。因为Django并不会在这个问题上给我们抛出异常。