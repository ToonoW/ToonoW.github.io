---
title: 将 JSON 转换成 SQL 脚本文件
date: 2018-04-02 10:25:07
tags:
- Python
- 工具脚本
categories:
- Python
---

现有单机 MongoDB 爬虫内容库存储网页上获取的内容，最大的 collection 记录条数为47万条数据。为了加速 MongoDB 上内容的使用速度，需要将 MongoDB 上部分字段存储一份到 MySQL 上，建立一个索引表，来达到页数统计和翻页的效果，在结构化数据库也更快速进行关键字段的检索，然后使用对应 MongoDB 的 _id 获取此行更详细的内容。这样来解决 MongoDB 中 collection 过大导致响应过慢的问题。

一开始的方案是直接编写脚本实时从 MongoDB 取数据，然后实时存入 MySQL。但是在实际操作的过程中会因为 MySQL 连接超时，无法完成数据转移。同时网络速度也不快，不稳定因素太多。之后选择先在 MongoDB 服务器进行部分字段的导出成 JSON 文件，并将 JSON 取回本地用脚本转换成 SQL 脚本的方案。这样子的好处有以下：

* 转换结果可见。相比之前的方案，能够看到所有转换完成的 SQL，当插入 MySQL 出错的时候可以定位并查找问题；
* 减少网络稳定性的影响；
* 处理速度更快。集中完成计算密集型工作，转换工作和网络 I/O 分离。



脚本依赖 `tqdm` 和 `pymysql` 库，需要提前安装。

下面先直接贴上脚本文件 `translate_json_to_sql.py`

```python
# -*- coding: utf-8 -*- 
import sys
import json
import logging

from tqdm import tqdm	# 可视进度条 https://github.com/tqdm/tqdm
from pymysql import escape_string	# 对字符串进行转义的函数


def main(filename, table_name):
    # 打开 JSON 文件
    with open(filename, 'r') as f:
        try:
            data = json.load(f)
        except json.JSONDecodeError:
            print('非合法JSON文件')
        else:
            # 创建或者覆盖 JSON 文件对应的 SQL 文件
            with open(filename + '.sql', 'w') as f2:
                # 使用 tqdm 生成器对 JSON 可迭代对象 data 进行迭代和进度反映
                for item in tqdm(data, desc='转换进度'):
                    sql = "insert into `{}` (`title`, `source`, `source_url`, `image_urls`, `_id`) values ('{}', '{}', '{}', '{}', '{}');\n"
                    try:
                        # 写入 SQL 语句到 SQL 文件
                        f2.write(sql.format(
                            table_name,
                            escape_string(item['title']),
                            escape_string(item['source']),
                            escape_string(item['source_url']),
                            escape_string(str(item['image_urls'])),
                            item['_id']['$oid'],
                        ))
                    except KeyError as e:
                        print(e)
                    except AttributeError as e:
                        print(e)


if __name__ == '__main__':
    try:
        # 传递命令行输入的两个参数 JSON 文件名和数据库表格名称到 main 函数
        main(sys.argv[1], sys.argv[2])
    except IndexError:
        logging.error('缺少参数。需要提供json文件名及数据库表格名称。')
```

在使用的时候输入命令

```shell
python translate_json_to_sql.py json文件名 对应的sql表格名称
```

