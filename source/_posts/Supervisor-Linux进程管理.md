---
title: 'Supervisor: Linux进程管理'
date: 2017-11-08 15:09:07
tags:
- Linux
- Supervisor
categories:
- Linux
---

[Supervisor](http://www.supervisord.org/) 是 Python 实现的成熟度的进程管理工具。方便启动关闭非后台运行的服务，并且服务程序意外退出的时候自动重启服务程序。

### 安装

Centos

`sudo yum install supervisor`

### 配置

软件的总配置在`/etc/supervisord.conf`，每个进程的配置文件都可以拆分开，放在`/etc/supervisord.d`目录下，注意需要是`.ini`后缀名。

假设我们编写一个 hubot 的配置文件，在`/etc/supervisord.d`目录下创建文件`hubot.ini`。

```ini
[program:hubot]
command=/home/wujinwen/hubot/bin/hubot --adapter slack
environment=HUBOT_SLACK_TOKEN=xoxb-2324175,HUBOT_SLACK_RTM_CLIENT_OPTS='{ "retryConfig": { "retries": 20 } }'
directory=/home/wujinwen/hubot
user=root
stdout_logfile=/home/wujinwen/hubot/log/hubot.log
redirect_stderr=true
```

进程 hubot 的名字定义在 `[program:hubot]`里。

* `command` 是启动 hubot 的命令；
* `environment`是环境变量的设置；
* `directory`是进程的当前目录；
* `stdout_logfile`是输出日志存放的路径；
* `redirect_stderr`是设置是否将错误信息直接输出到日志文件中；
* `user`是运行进程的用户。

假若是使用了 virtualenv 这个虚拟环境，`commend`的程序执行路径需要填写虚拟环境中的路径，例如`/home/wujinwen/.pyenv/versions/envRS/bin/celery`。

### 使用

当我们编写好配置文件之后，就可以重启 supervisor 然后启动我们的服务了。

```shell
# 重启 supervisor
supervisorctl reload

# 手动开启我们的 hubot 服务 （正常情况下重启 supervisorctl 就会启动所有允许自动执行的服务了）
supervisorctl start hubot

# 重启 hubot 的进程
supervisorctl restart hubot

# 停止 hubot 的进程
supervisorctl stop hubot
```

当修改了配置之后可以单独重启部分进程生效

```ini
# 读取更改后的配置，检查变动及是否存在错误
supervisorctl reread

# 更新配置，重启改动了配置的进程
supervisorctl update
```

### 进阶配置与使用 - 分组

分组利于我们管理同一项目的几个进程，在很多情况下好几个进程需要同时进行启动或者关闭操作。有了分组的概念，我们就不需要独单地去启动每一个进程了，可以以分组为单位对进程组启动关闭等操作。

这里粘贴一个 Gunicorn + Django + Celery 的示例配置

```ini
[program:EB-celery-beat]
command=/home/wujinwen/Codes/EventsBroker/venv/bin/celery -A EventsBroker beat -l info
directory=/home/wujinwen/Codes/EventsBroker
user=wujinwen
stdout_logfile=/home/wujinwen/Codes/EventsBroker/logs/celery-beat.log
redirect_stderr=true


[program:EB-celery-worker]
command=/home/wujinwen/Codes/EventsBroker/venv/bin/celery -A EventsBroker worker -l info
directory=/home/wujinwen/Codes/EventsBroker
user=wujinwen
stdout_logfile=/home/wujinwen/Codes/EventsBroker/logs/celery-worker.log
redirect_stderr=true


[program:EB-gunicorn]
command=/home/wujinwen/Codes/EventsBroker/venv/bin/gunicorn --worker-class=gevent EventsBroker.wsgi:application  -b 127.0.0.1:8013 --reload
directory=/home/wujinwen/Codes/EventsBroker
user=wujinwen
stdout_logfile=/home/wujinwen/Codes/EventsBroker/logs/gunicorn.log
redirect_stderr=true


[group:EventsBroler]
programs=EB-celery-beat,EB-celery-worker,EB-gunicorn
```

注意我们在最后有一行是 `[group:EventsBroler]` 这里我们设定了一个分组 `EventsBroker`，包括了三个进程，进程名之间用逗号分隔。（还有优先级设置可选，详细查看[官方文档](http://supervisord.org/configuration.html#group-x-section-settings)）

##### 使用

使用方法与进程的启动关闭类似。

```ini
# 启动分组内所有进程
supervisorctl start 分组名:*
```

### 参考

> [Linux后台进程管理利器：supervisor](https://www.liaoxuefeng.com/article/0013738926914703df5e93589a14c19807f0e285194fe84000)

