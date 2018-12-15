---
title: Superset踩雷日志
date: 2017-09-08 15:35:47
tags:
- Superset
- Python
- 数据可视化
categories:
- Python
- 数据可视化
---

## 生成图表时出错`unorderable types: str() > int()`

> Superset 0.19.1

```shell
Traceback (most recent call last):
  File "/Users/wujinwen/Codes/Python/CRAnalysis/incubator-superset/superset/viz.py", line 253, in get_payload
    data = self.get_data(df)
  File "/Users/wujinwen/Codes/Python/CRAnalysis/incubator-superset/superset/viz.py", line 924, in get_data
    df = self.process_data(df)
  File "/Users/wujinwen/Codes/Python/CRAnalysis/incubator-superset/superset/viz.py", line 867, in process_data
    values=fd.get('metrics'))
  File "/Users/wujinwen/.pyenv/versions/35env/lib/python3.5/site-packages/pandas-0.20.2-py3.5-macosx-10.12-x86_64.egg/pandas/core/reshape/pivot.py", line 160, in pivot_table
    table = table.sort_index(axis=1)
  File "/Users/wujinwen/.pyenv/versions/35env/lib/python3.5/site-packages/pandas-0.20.2-py3.5-macosx-10.12-x86_64.egg/pandas/core/frame.py", line 3243, in sort_index
    labels = labels._sort_levels_monotonic()
  File "/Users/wujinwen/.pyenv/versions/35env/lib/python3.5/site-packages/pandas-0.20.2-py3.5-macosx-10.12-x86_64.egg/pandas/core/indexes/multi.py", line 1240, in _sort_levels_monotonic
    indexer = lev.argsort()
  File "/Users/wujinwen/.pyenv/versions/35env/lib/python3.5/site-packages/pandas-0.20.2-py3.5-macosx-10.12-x86_64.egg/pandas/core/indexes/base.py", line 2091, in argsort
    return result.argsort(*args, **kwargs)
TypeError: unorderable types: str() > int()
```

因为在实际情况中部分维度（字段）的某些行存在值为空的情况，这就会导致以上错误。

这种情况出现在生成图表的时候，原因为类型不统一。猜测Superset默认将空值填充为0，而不为空的值可能为`TEXT`，也可能为其他数据类型。当不为数据类型为`TEXT`的时候就会出现以上错误`TypeError: unorderable types: str() > int()`，即为整型的0与字符串进行排序比较。

一开始会疑惑为什么Superset作者没有将空值对应数据类型填充'NULL'和0之类的，后来翻看了issue记录才知道他是觉得并非所有用户愿意这样直接的替换，以致使用者的误会，所以一直没有对这个问题进行解决。原话如下。

> I'm not sure whether we want to systematically replace `NULL` with `<None>` and empties with `0`. I understand the use case but people may not want that in their existing charts....

根据我的理解，假如某个维度为数值类型，那么如果空值被填充为0，那么很容易引起误会，用户并不知道0到底是原始数据还是被填充上的占位值。同样的，假如某个维度为字符类型，空值被填充为'NULL'同样有可能引起用户的误会。

所以**临时的解决方法**是为存在空值的维度添加`Filters`，将空值过滤掉即可正常使用。需要注意的是，在点击增加`Filters`之后选择维度和逻辑关系，但是**不需要填写和点击过滤值的内容框**。