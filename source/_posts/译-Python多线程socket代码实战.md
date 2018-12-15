---
title: 译-Python多线程socket代码实战
date: 2017-01-12 15:32:06
tags:
- Python
- Socket
categories:
- Python

---

## 说在前面

因为中文太难搜索到合适的 socket 的实用编程案例，能找到的都是简单使用 socket 居多，水分比较大。而且在做的是 GUI 编程，所以特意找到了[这篇文章](http://eli.thegreenplace.net/2011/05/18/code-sample-socket-client-thread-in-python)来学习，感觉非常实用。翻译没有把握的部分我都给出了原文，希望能够给出更好的翻译，共勉。

## 正文

当我们创建GUI图形界面程序的时候经常需要用到网络编程去获取一些实用数据，而我们通常遇到的绊脚石就是怎么将 GUI 结合 I/O 编程，无论是 HTTP 请求，还是 RPC 协议，简单的 socket 通信还是连续的端口(serial port)，这些问题让我们没那么容易将 I/O 编程与 GUI 很好地结合在一起。没有人希望在他的 socket 接收程序的时候图形界面被“冻结”起来。

有许多方法能解决以上问题，这里列举两个常用的方法：

1. 在单独的线程进行 I/O 操作
2. 利用异步 I/O 和回调方法，将 I/O 操作整合进 GUI 的 event loop 中

在我看来，第一种方法是两者中最简便的，而且也是我最常用的方法。这里我将写一个简单的代码例子，它实现了 socket 多线程。虽然这个类通常情况可以解决很多场景的问题，但是我更愿意将它看作一种设计模式，而不是一个万能的黑盒(black-box)。网络编程代码依赖非常多因素，而且这个例子很容易被修改成适应各种场景(Networking code tends to depend on a lot of factors, and it's easy to modify this sample to various scenarios)。例如，这是一个客户端的代码，但是也很容易重新实现为相似的服务器程序。话不多说，以下是代码：

```python
import socket
import struct
import threading
import Queue


class ClientCommand(object):
    """ 为客户端线程服务的命令
        每个命令类型有与它相关的数据类型:

        CONNECT:    (host, port) tuple(元组)
        SEND:       Data string
        RECEIVE:    None
        CLOSE:      None
    """
    CONNECT, SEND, RECEIVE, CLOSE = range(4)

    def __init__(self, type, data=None):
        self.type = type
        self.data = data


class ClientReply(object):
    """ 来自客户端线程的回复
        每个回复有与它相关的数据类型:

        ERROR:      The error string
        SUCCESS:    与命令类型有关，RECEIVE 是接收到的数据字符串(data string)，而					 其他类型为 None
    """
    ERROR, SUCCESS = range(2)

    def __init__(self, type, data=None):
        self.type = type
        self.data = data


class SocketClientThread(threading.Thread):
    """ 实现 threading. Thread 接口(start, join ,等等) 还可以通过属性 cmd_q 队列
    	来控制。消息回复被放在属性 reply_q 队列中。
    """
    def __init__(self, cmd_q=None, reply_q=None):
        super(SocketClientThread, self).__init__()
        self.cmd_q = cmd_q or Queue.Queue()
        self.reply_q = reply_q or Queue.Queue()
        self.alive = threading.Event()
        self.alive.set()
        self.socket = None

        self.handlers = {
            ClientCommand.CONNECT: self._handle_CONNECT,
            ClientCommand.CLOSE: self._handle_CLOSE,
            ClientCommand.SEND: self._handle_SEND,
            ClientCommand.RECEIVE: self._handle_RECEIVE,
        }

    def run(self):
        while self.alive.isSet():
            try:
                # Queue.get with timeout to allow checking self.alive
                cmd = self.cmd_q.get(True, 0.1)
                self.handlers[cmd.type](cmd)
            except Queue.Empty as e:
                continue

    def join(self, timeout=None):
        self.alive.clear()
        threading.Thread.join(self, timeout)

    def _handle_CONNECT(self, cmd):
        try:
            self.socket = socket.socket(
                socket.AF_INET, socket.SOCK_STREAM)
            self.socket.connect((cmd.data[0], cmd.data[1]))
            self.reply_q.put(self._success_reply())
        except IOError as e:
            self.reply_q.put(self._error_reply(str(e)))

    def _handle_CLOSE(self, cmd):
        self.socket.close()
        reply = ClientReply(ClientReply.SUCCESS)
        self.reply_q.put(reply)

    def _handle_SEND(self, cmd):
        header = struct.pack('<L', len(cmd.data))
        try:
            self.socket.sendall(header + cmd.data)
            self.reply_q.put(self._success_reply())
        except IOError as e:
            self.reply_q.put(self._error_reply(str(e)))

    def _handle_RECEIVE(self, cmd):
        try:
            header_data = self._recv_n_bytes(4)
            if len(header_data) == 4:
                msg_len = struct.unpack('<L', header_data)[0]
                data = self._recv_n_bytes(msg_len)
                if len(data) == msg_len:
                    self.reply_q.put(self._success_reply(data))
                    return
            self.reply_q.put(self._error_reply('Socket closed prematurely'))
        except IOError as e:
            self.reply_q.put(self._error_reply(str(e)))

    def _recv_n_bytes(self, n):
        """ 方便的方法，为了从 self.socket(假定是打开而且已经连接)准确接收 n byte的			  数据
        """
        data = ''
        while len(data) < n:
            chunk = self.socket.recv(n - len(data))
            if chunk == '':
                break
            data += chunk
        return data

    def _error_reply(self, errstr):
        return ClientReply(ClientReply.ERROR, errstr)

    def _success_reply(self, data=None):
        return ClientReply(ClientReply.SUCCESS, data)
```

`SocketClientThread`是这里的主要类。它是 Python 线程，可以被启动和终止(joined),而且外界通过上面定义的命令类`ClientCommand`和 回复类 `ClientReply`进行交互。`ClientCommand` 和 `ClientReply`是结构简单的数据结构类，用来概括各种命令和回复。

以上代码很简单，但是同时示范了好几种 Python 多线程和网络编程的设计模式。这里做简单的介绍比较有意思的点，以下内容和顺序无关:

- 标准的`Queue.Queue`被用作在用户代码和线程之间的数据传递。`Queue`是 Python 程序员工具箱里面很棒的一个工具 - 我总是用它去解耦多线程代码。编写多线程代码最大的不同点就是要去保护共享的数据。`Queue`本质上改变了数据传递的模式，它简单使用而且很安全。

  你会注意到`SocketClientThread`用到了两个队列，一个用来从主线程获取命令，另一个为了传递回复。这是一个常用的风格，而且在很多场景可以发挥很好的作用。

- 一般来说，在 Python 你很难强制杀死一个线程。如果你需要手动终止一个线程，线程他们必须同意被终止。`SocketClientThread`的`alive`属性示范了一种常用而且安全的方式去实现终止线程。`alive`是一个`threading.Event` - 一个线程安全的 flag，它能在主线程调用`alive.clear()`被清除（它在`join`方法中执行）。交互线程(communication thread)时不时去检查 flag，如果flag为否了，线程会优雅地退出。

  这里有非常重要的实现细节。注意一下线程的`run`方法如何被重新实现。当`alive`是`set`的时候循环(loop)会不断执行，循环不会被打破。方法`get(True, 0.1)`会每隔0.1秒去命令队列拉取一次命令，这就意味着这个行为会阻塞0.1秒。这样子做有两个好处：一方面，它不会无限期被阻塞，至少0.1秒会通过一次循环而且检查一次`alive`有没有被清除。另一方面，因为阻塞0.1秒，所以线程在等待命令的时候CPU不会高负荷运行。事实上，它的CPU负荷是可以忽略不计的。注意这点，在线程的socket连接的`recv`(阻塞的方法)还没有数据进入的时候，线程可以保持阻塞并且拒绝被结束。

- `SocketClientThread`使用TCP socket连接，它会高保真地传送所有数据，包括很大的未知大小的数据。为了让对方知道我们需要传送的数据的起始和技术，这就需要我们以某种方式去界定消息的长度信息。我通常用长度信息前缀的技术。当一个穿数据在发送之前，数据的长度就以4byte的包发过去。对方首先会解析先接收到的4byte去获取数据的长度，然后在按照知道的长度去接收剩下的数据。

这个有个简单的[例子](https://github.com/eliben/code-for-blog/tree/master/2011/socket_client_thread_sample)去示范怎么使用`SocketClientThread`，例子也包含了一个利用PyQt库实现的简单的GUI。这个GUI利用`SocketClientThread`线程去连接服务程序(默认地址端口是`localhost:50007`)，发送`hello`然后等待回复。在大部分时间，GUI上会显示一个旋转动画的图标去证明GUI的主线程没有被`SocketClientThread`线程阻塞。为了达到这个效果，GUI使用了有趣的控件 - timer，它会周期性地通过`reply_q.get(block=False)`检查`SocketClientThread`线程放置在回复队列中的数据。结合timer和非阻塞的`get`方法，我们就可以有效地在GUI和子线程之间交流了。

我希望以上的代码例子会对大家有用处。

P.S.代码都在[这儿](https://github.com/eliben/code-for-blog/tree/master/2011/socket_client_thread_sample)