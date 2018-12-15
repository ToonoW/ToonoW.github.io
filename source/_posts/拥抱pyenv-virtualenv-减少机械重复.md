---
title: '拥抱pyenv+virtualenv, 减少机械重复'
date: 2017-11-13 14:22:02
tags:
- Python
- pyenv
- virtualenv
categories:
- Python
---

我是一名 VSCode 的深度使用者，几乎所有的编码工作都是发生在 VSCode。所以越来越重视在 VSCode 编码中每一个环节的效率。

而 Python 则是我的主要生产工具，VSCode 通过插件的帮助，对 Python 的支持已经很好了。但是在我的使用中，美中不足的是每次新建「终端」之后都需要执行一句 `source venv/bin/activate`。虽然随着输入次数越来越多，写入这句命令已经可以在一两秒内解决，但是这种机械重复的工作在我专职写 Python 半年之后真的受够了，下定决心使用能够解决这个问题的 pyenv 。

[pyenv](https://github.com/pyenv/pyenv) 使你轻松在多个版本的 python 之间切换。pyenv 是 fork [rbenv](https://github.com/rbenv/rbenv) 和 [ruby-build](https://github.com/rbenv/ruby-build) 的，是它们在 python 上的实现。另一个重要的工具是 [virtualenv](https://virtualenv.pypa.io/en/stable/)，它是一个可以方便创建独立、干净 python 环境的工具，我们这里是使用它在 pyenv 下的插件。

## 安装 pyenv

pyenv 的[安装步骤](https://github.com/pyenv/pyenv#installation), 以下是对其的简要翻译。

### 自动安装工具

请浏览 GitHub 项目：https://github.com/pyenv/pyenv-installer

### 从 GitHub 下载安装

这样的下载方式你会得到代码的最新版本，而且你也很容易将你对软件的修改提交到 GitHub 仓库上游。

1. **在你想要安装 pyenv 的地方检出其 GitHub 仓库。**一个比较好的地方是 `$HOME/.pyenv`，你的个人目录（你也可以在其他地方检出）。

   ```shell
   $ git clone https://github.com/pyenv/pyenv.git ~/.pyenv
   ```

2. **设置环境变量** `PYENV_ROOT` 去指向你存放刚检出的 pyenv 的路径，增加 `$PYENV_ROOT/bin` 到 `$PATH` 让你的 `pyenv` 可以在命令行直接输入此命令。

   ```shell
   $ echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bash_profile
   $ echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bash_profile
   ```

3. **增加 `pyenv init` 到你的终端配置**去启用 shims 和自动补全。请确保 `eval "$(pyenv init -)"` 在配置文件的末尾，因为这个语句在初始化期间使用到了 `PATH`。

   ```shell
   $ echo -e 'if command -v pyenv 1>/dev/null 2>&1; then\n  eval "$(pyenv init -)"\nfi' >> ~/.bash_profile
   ```

4. **重启你的终端让改动生效。**你可以开始使用 pyenv 。

   ```shell
   $ exec "$SHELL"
   ```

5. 安装各种版本的 python 到 **$(pyenv root)/versions**。例如，下载和安装 Python 2.7.8，执行：

   ```shell
   $ pyenv install 2.7.8 
   ```

### MacOS 利用 Homebrew 安装

如果你使用 MacOS 的话，你也可以使用 Homebew 安装 pyenv。

```shell
$ brew update
$ brew install pyenv
```

   

更多自定义的配置请参阅 GitHub 的 [README 文档](https://github.com/pyenv/pyenv#advanced-configuration)

## 安装 pyenv-virtualenv

以下为 pyenv-virtualenv [安装步骤](https://github.com/pyenv/pyenv-virtualenv#installation)的简要翻译

### 作为 pyenv 插件安装

接下来的操作会将最新版的 pyenv-virtualenv 安装到 `$(pyenv root)/plugins/pyenv-virtualenv` 目录。

**重要提示：**如果你安装 pyenv 到非默认的目录，确保你克隆本仓库到你 pyenv 安装目录的 `plugins` 文件夹下。

1. **检出 pyenv-virtualenv 到你的插件目录**

   ```shell
   $ git clone https://github.com/pyenv/pyenv-virtualenv.git $(pyenv root)/plugins/pyenv-virtualenv
   ```

2. （可选）**增加 pyenv virtualenv-init 到你的终端配置中**，自动激活 virtualenvs。这虽然是可选的，但是非常的有用。

   ```shell
   $ echo 'eval "$(pyenv virtualenv-init -)"' >> ~/.bash_profile
   ```

3. **重启你的终端以启用 pyenv-virtualenv**

   ```shell
   exec "$SHELL"
   ```

### 通过 Homebrew 安装

如果你使用的是 Homebrew 来管理你的软件，那么你可以通过它安装 `pyenv-virtualenv`。

```shell
$ brew install pyenv-virtualenv
```

安装结束之后，你还需要将下面两条语句添加到终端配置文件 `~/.bash_profile` 中

```shell
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
```

