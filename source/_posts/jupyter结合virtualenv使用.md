---
title: jupyter 结合 virtualenv 使用，增加 kernel
date: 2018-02-05 16:34:06
tags:
- Python
categories:
- Python
---

利用 jupyter 帮助写一些实验性的代码非常地方便，很容易就能测试效果。并且在 jupyter 可以上编写的代码会被持久化，可以当作简单的代码笔记本使用。也可以用来导出其他格式，交付可视的实验结果。   

jupyter 结合 virtualenv 使用的方法并不是那么直接，需要在 jupyter 的安装目录下去增加虚拟环境对应的配置文件。   

>  以下的记录中使用的虚拟环境工具是 pipenv ，但是同样适用于其他 virtualenv 虚拟环境工具。我们只需要 virtualenv 中 python 文件的准确路径。

## 进行之前

* 在此默认你已经掌握 virtuanlev 的基本使用；
* 你已经在全局环境下安装了 jupyter；
* 会基本使用 pip;

## 正文

### 1. 首先得知道虚拟环境的路径

```shell
# 获取虚拟环境的路径（使用virtualenv的话，只需要清楚虚拟环境文件中的路径，以备用）
> pipenv --venv
/Users/jinwen/.local/share/virtualenvs/ProcessLogo-Yd87wy1W
```

### 2. 找到 jupyter 配置文件的路径

```Shell
# 利用 find 命令在磁盘查找 jupyter 的安装路径
> sudo find / -name jupyter
/Library/Frameworks/Python.framework/Versions/3.6/bin/jupyter
/Library/Frameworks/Python.framework/Versions/3.6/etc/jupyter
/Library/Frameworks/Python.framework/Versions/3.6/share/jupyter

# 进入配置文件所在的目录
> cd /Library/Frameworks/Python.framework/Versions/3.6/share/jupyter/kernels

# 查看现有的配置文件的文件夹
> ls
python3
```

经过上面的操作，我们已经找到了存放配置文件的目录了。

### 3. 创建我们需要的 virtualenv 配置

```Shell
# 复制一份配置文件夹修改成我们需要的配置
> cp -r python3 process_logo	# 这里的 process_logo 需要改成你需要的配置名
```

利用我们第一步操作知道的虚拟环境路径，补充完整为虚拟环境中 python 的路径，并且用此路径替换 `kernel.json` 中 `argv` 对应 value 中的 python。而其中的 `display_name` 为显示的名称，可自行命名，但不建议与现存命名的重复。

`kernel.json` 在我这里替换的结果预览为：

```Json
{
 "argv": [
  "/Users/jinwen/.local/share/virtualenvs/ProcessLogo-Yd87wy1W/bin/python",
  "-m",
  "ipykernel_launcher",
  "-f",
  "{connection_file}"
 ],
 "language": "python",
 "display_name": "process logo"
}
```

### 4. 还需要在虚拟环境中装 jupyter

因为 kernel 的激活依赖一些 jupyter 的模块，所以还需要在虚拟环境中安装一份 jupyter，否则没办法启动 kernel。

```shell
# 在你的虚拟环境中使用 pip 等工具安装 jupyter
> /Users/jinwen/.local/share/virtualenvs/ProcessLogo-Yd87wy1W/bin/python -m pip install jupyter
```

   

至此就可以进入 jupyter 去使用新增加的 kernel了，在对应的虚拟环境中安装过的模块都可以 import 并且使用。