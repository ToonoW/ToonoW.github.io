---
title: linux命令记录
date: 2018-01-06 23:32:21
tags:
- Linux
---

```bash
# 通过ssh建立代理隧道
ssh -N -D 127.0.0.1:本地监听的端口 用户名@远端服务器地址

# MongoDB 导出json
mongoexport [-h host_ip] [--port port_num]  -d database_name -c collection_name [-q query_string] --type json -o local_store_path

# 让 pyenv 使用国内镜像安装 Python
# 修改开头的版本号来指定安装的版本
v=3.6.5|wget http://mirrors.sohu.com/python/$v/Python-$v.tar.xz -P ~/.pyenv/cache/;pyenv install $v
```

