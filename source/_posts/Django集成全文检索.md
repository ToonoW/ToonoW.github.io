---
title: Django集成全文检索
date: 2017-10-27 17:13:48
tags:
- Python
- Django
categories:
- Python
---

因为有高效检索数据库的需求，就选择了**Elasticsearch**(简称ES)作为搜索引擎。如果对ES不熟悉，直接用官网提供的客户端的话还是有一定的难度。所以为了省时间我使用了haystack，他可以连接不少的搜索引擎，其中就包括ES和whoosh。

按照haystack官方文档教程一路配置下去都没问题，但是巧在我models中的字段存在空值，所以需要在`search_indexes.py`文件中定义的模型字段添加属性`null=True`，即

```python
class SubtitleIndex(indexes.SearchIndex, indexes.Indexable):
    ...
    pub_date = indexes.DateTimeField(model_attr='pub_date', null=True)
```

顺带一说这个错误是

```python
haystack.exceptions.SearchFieldError: The model '<Subtitle: 3 idiots>' has an empty model_attr 'pub_date' and doesn't allow a default or null value.
```

参考自[StackOverflow](http://stackoverflow.com/a/22695639/1936697)

