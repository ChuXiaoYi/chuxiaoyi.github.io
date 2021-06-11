---
title: tornado（一）——基础介绍
date: 2019-02-12 10:40:23
tags: tornado tornado
categories: python
---


参考文档：

- <https://docs.pythontab.com/tornado/introduction-to-tornado/ch1.html#ch1-2-1-1>
- <https://www.tornadoweb.org/en/stable/guide/intro.html>

开发环境：

<!--more-->

- python3.7
- tornado5.1

# Tornado是什么？

Tornado是一个Python Web框架和异步网络库，最初是在FriendFeed上开发的。通过使用非阻塞网络I / O，Tornado可以扩展到数万个开放连接，使其成为长轮询， WebSockets和其他需要与每个用户建立长期连接的应用程序的理想选择 。

不同于那些最多只能达到10,000个并发连接的传统网络服务器，Tornado在设计之初就考虑到了性能因素，旨在解决[C10K](#1)问题，这样的设计使得其成为一个拥有非常高性能的框架。此外，它还拥有处理安全性、用户验证、社交网络以及与外部服务（如数据库和网站API）进行异步交互的工具。

tornado大致分为四个部分：

1.  [tornado.web](http://tornado.web)
2.  tornado.ioloop
3.  tornado.httpserver
4.  tornado.gen\(当前版本可以使用`async def`代替\)

# 安装

```
pip install tornado
```

# 简单入门

我们从简单的栗子开始，逐渐深入了解tornado吧～

### Hello Tornado

```python
# -*- coding: utf-8 -*-
# --------------------------------------
#       @Time    : 2019/1/28 上午11:43
#       @Author  : cxy =.= 
#       @File    : hello.py
#       @Software: PyCharm
#       @Desc    : 
# --------------------------------------
import tornado.httpserver
import tornado.ioloop
import tornado.options
import tornado.web

from tornado.options import define, options

define("port", default=8000, help="run on the given port", type=int)


class IndexHandler(tornado.web.RequestHandler):
    def get(self):
        greeting = self.get_argument('greeting', 'Hello')
        self.write(greeting + ', friendly user!')


if __name__ == "__main__":
    tornado.options.parse_command_line()
    app = tornado.web.Application(
        handlers=[
            (r"/", IndexHandler)
        ]
    )
    http_server = tornado.httpserver.HTTPServer(app)
    http_server.listen(options.port)
    tornado.ioloop.IOLoop.instance().start()
```

编写一个Tornado应用中最多的工作是定义类继承Tornado的RequestHandler类。在这个例子中，我们创建了一个简单的应用，在给定的端口监听请求，并在根目录（"/"）响应请求。

你可以在命令行里尝试运行这个程序以测试输出：

```
python hello.py --port=8000
```

现在你可以在浏览器中打开http://localhost:8000, 或者打开另一个终端窗口使用curl测试我们的应用:  
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019021211242430.png)  
此时，你会在终端中看到：  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190212112536829.png)

---

**让我们把这个例子分成小块，逐步分析它们：**

```python
import tornado.httpserver
import tornado.ioloop
import tornado.options
import tornado.web
```

在程序的最顶部，我们导入了一些Tornado模块。虽然Tornado还有另外一些有用的模块，但在这个例子中我们必须至少包含这四个模块。

```python
from tornado.options import define, options
define("port", default=8000, help="run on the given port", type=int)
```

Tornado包括了一个有用的模块（`tornado.options`）来从命令行中读取设置。我们在这里使用这个模块指定我们的应用监听HTTP请求的端口。它的工作流程如下：如果一个与`define`语句中同名的设置在命令行中被给出，那么它将成为全局`options`的一个属性。如果用户运行程序时使用了`--help`选项，程序将打印出所有你定义的选项以及你在define函数的help参数中指定的文本。如果用户没有为这个选项指定值，则使用`default`的值进行代替。Tornado使用`type`参数进行基本的参数类型验证，当不合适的类型被给出时抛出一个异常。因此，我们允许一个整数的port参数作为options.port来访问程序。如果用户没有指定值，则默认为`8000`。

```python
class IndexHandler(tornado.web.RequestHandler):
    def get(self):
        greeting = self.get_argument('greeting', 'Hello')
        self.write(greeting + ', friendly user!')
```

这是Tornado的请求处理函数类。当处理一个请求时，Tornado将这个类实例化，并调用与HTTP请求方法所对应的方法。在这个例子中，我们只定义了一个get方法，也就是说这个处理函数将对HTTP的GET请求作出响应。我们稍后将看到实现不止一个HTTP方法的处理函数。

```python
greeting = self.get_argument('greeting', 'Hello')
```

Tornado的`RequestHandler`类有一系列有用的内建方法，包括`get_argument`，我们在这里从一个querystring中取得参数greeting的值。（如果这个参数没有出现在querystring中，Tornado将使用`get_argument`的第二个参数作为默认值。）

```python
self.write(greeting + ', friendly user!')
```

RequestHandler的另一个有用的方法是`write`，它以一个字符串作为函数的参数，并将其写入到HTTP响应中。在这里，我们使用请求中greeting参数提供的值赋值到greeting中，并写回到响应中。

```python
if __name__ == "__main__":
    tornado.options.parse_command_line()
    app = tornado.web.Application(
        handlers=[
            (r"/", IndexHandler)
        ]
    )
```

这是真正使得Tornado运转起来的语句。首先，我们使用Tornado的`options`模块来解析命令行。然后我们创建了一个Tornado的`Application`类的实例。传递给Application类`__init__`方法的最重要的参数是`handlers`。它告诉Tornado应该用哪个类来响应请求。

这里的参数handlers非常重要，它应该是一个元组组成的列表，其中每个元组的第一个元素是一个用于匹配的正则表达式，第二个元素是一个RequestHanlder类。在hello.py中，我们只指定了一个`正则表达式-RequestHanlder`对，但你可以按你的需要指定任意多个。

Tornado在元组中使用正则表达式来匹配HTTP请求的路径。（这个路径是URL中主机名后面的部分，不包括查询字符串和碎片。）Tornado把这些正则表达式看作已经包含了行开始和结束锚点（即，字符串"/“被看作为”\^/\$"）。

如果一个正则表达式包含一个捕获分组（即，正则表达式中的部分被括号括起来），匹配的内容将作为相应HTTP请求的参数传到RequestHandler对象中。我们将在下个例子中看到它的用法。

```python
http_server = tornado.httpserver.HTTPServer(app)
http_server.listen(options.port)
tornado.ioloop.IOLoop.instance().start()
```

从这里开始的代码将会被反复使用：一旦`Application`对象被创建，我们可以将其传递给Tornado的`HTTPServer`对象，然后使用我们在命令行指定的端口进行监听（通过options对象取出。）最后，在程序准备好接收HTTP请求后，我们创建一个Tornado的`IOLoop`的实例。

### 关于RequestHandler的更多知识

```python
# -*- coding: utf-8 -*-
# ---------------------------------------------
#       @Time    : 2019/2/12 上午11:33
#       @Author  : cxy =.= 
#       @File    : string_service.py
#       @Software: PyCharm
#       @Desc    : 
# ---------------------------------------------
import textwrap

import tornado.httpserver
import tornado.ioloop
import tornado.options
import tornado.web

from tornado.options import define, options

define("port", default=8000, help="run on the given port", type=int)


class ReverseHandler(tornado.web.RequestHandler):
    def get(self, input):
        self.write(input[::-1])


class WrapHandler(tornado.web.RequestHandler):
    def post(self):
        text = self.get_argument('text')
        width = self.get_argument('width', 40)
        self.write(textwrap.fill(text, int(width)))


if __name__ == "__main__":
    tornado.options.parse_command_line()
    app = tornado.web.Application(
        handlers=[
            (r"/reverse/(\w+)", ReverseHandler),
            (r"/wrap", WrapHandler)
        ]
    )
    http_server = tornado.httpserver.HTTPServer(app)
    http_server.listen(options.port)
    tornado.ioloop.IOLoop.instance().start()
```

如同运行第一个例子，你可以在命令行中运行这个例子使用如下的命令：

```
python string_service.py --port=8000
```

这个程序是一个通用的字符串操作的Web服务端基本框架。到目前为止，你可以用它做两件事情。

其一，到/reverse/string的GET请求将会返回URL路径中指定字符串的反转形式。  
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019021212252356.png)

其二，到/wrap的POST请求将从参数text中取得指定的文本，并返回按照参数width指定宽度装饰的文本。下面的请求指定一个没有宽度的字符串，所以它的输出宽度被指定为程序中的get\_argument的默认值40个字符。  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190212122514506.png)

`string_service.py`和`hello.py`示例代码中大部分是一样的。让我们关注那些新的代码。首先，让我们看看传递给Application构造函数的`handlers`参数的值：

```python
app = tornado.web.Application(
        handlers=[
            (r"/reverse/(\w+)", ReverseHandler),
            (r"/wrap", WrapHandler)
        ]
    )
```

在上面的代码中，Application类在`handlers`参数中实例化了两个RequestHandler类对象。第一个引导Tornado传递路径匹配下面的正则表达式的请求：

```
/reverse/(\w+)
```

正则表达式告诉Tornado匹配任何以字符串`/reverse/`开始并紧跟着一个或多个字母的路径。括号的含义是让Tornado保存匹配括号里面表达式的字符串，并将其作为请求方法的一个参数传递给RequestHandler类。让我们检查ReverseHandler的定义来看看它是如何工作的：

```
class ReverseHandler(tornado.web.RequestHandler):
    def get(self, input):
        self.write(input[::-1])
```

你可以看到这里的`get`方法有一个额外的参数`input`。这个参数将包含匹配处理函数正则表达式第一个括号里的字符串。（如果正则表达式中有一系列额外的括号，匹配的字符串将被按照在正则表达式中出现的顺序作为额外的参数传递进来。）

现在，让我们看一下`WrapHandler`的定义：

```python
class WrapHandler(tornado.web.RequestHandler):
    def post(self):
        text = self.get_argument('text')
        width = self.get_argument('width', 40)
        self.write(textwrap.fill(text, int(width)))
```

WrapHandler类处理匹配路径为`/wrap`的请求。这个处理函数定义了一个post方法，也就是说它接收HTTP的POST方法的请求。

我们之前使用RequestHandler对象的`get_argument`方法来捕获请求查询字符串的的参数。同样，我们也可以使用相同的方法来获得POST请求传递的参数。（Tornado可以解析URLencoded和multipart结构的POST请求）。一旦我们从POST中获得了文本和宽度的参数，我们使用Python内建的textwrap模块来以指定的宽度装饰文本，并将结果字符串写回到HTTP响应中。

### HTTP状态码

http的状态码我们都十分熟悉，你可以使用RequestHandler类的`set_status()`方法显式地设置HTTP状态码。然而，你需要记住在某些情况下，Tornado会自动地设置HTTP状态码。下面是一个常用情况的纲要：

- **404 Not Found**  
  Tornado会在HTTP请求的路径无法匹配任何RequestHandler类相对应的模式时返回404（Not Found）响应码。

- **400 Bad Request**  
  如果你调用了一个没有默认值的get\_argument函数，并且没有发现给定名称的参数，Tornado将自动返回一个400（Bad Request）响应码。

- **405 Method Not Allowed**  
  如果传入的请求使用了RequestHandler中没有定义的HTTP方法（比如，一个POST请求，但是处理函数中只有定义了get方法），Tornado将返回一个405（Methos Not Allowed）响应码。

- **500 Internal Server Error**

- 当程序遇到任何不能让其退出的错误时，Tornado将返回500（Internal Server Error）响应码。你代码中任何没有捕获的异常也会导致500响应码。

- **200 OK**  
  如果响应成功，并且没有其他返回码被设置，Tornado将默认返回一个200（OK）响应码。

当上述任何一种错误发生时，Tornado将默认向客户端发送一个包含状态码和错误信息的简短片段。如果你想使用自己的方法代替默认的错误响应，你可以重写`write_error`方法在你的RequestHandler类中。比如，以下代码是hello.py示例添加了常规的错误消息的版本。

```python
# -*- coding: utf-8 -*-
# --------------------------------------
#       @Time    : 2019/1/28 上午11:43
#       @Author  : cxy =.= 
#       @File    : hello-errors.py
#       @Software: PyCharm
#       @Desc    : 
# --------------------------------------
import tornado.httpserver
import tornado.ioloop
import tornado.options
import tornado.web

from tornado.options import define, options

define("port", default=8000, help="run on the given port", type=int)


class IndexHandler(tornado.web.RequestHandler):
    def get(self):
        greeting = self.get_argument('greeting', 'Hello')
        self.write(greeting + ', friendly user!')

    def write_error(self, status_code, **kwargs):
        self.write("Gosh darnit, user! You caused a %d error." % status_code)

if __name__ == "__main__":
    tornado.options.parse_command_line()
    app = tornado.web.Application(
        handlers=[
            (r"/", IndexHandler)
        ]
    )
    http_server = tornado.httpserver.HTTPServer(app)
    http_server.listen(options.port)
    tornado.ioloop.IOLoop.instance().start()
```

当我们尝试一个POST请求时，会得到下面的响应。一般来说，我们应该得到Tornado默认的错误响应，但因为我们覆写了`write_error`，我们会得到不一样的东西：  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190212122501749.png)

# 扩展

### C10K问题

基于线程的服务器，如Apache，为了传入的连接，维护了一个操作系统的线程池。Apache会为每个HTTP连接分配线程池中的一个线程，如果所有的线程都处于被占用的状态并且尚有内存可用时，则生成一个新的线程。尽管不同的操作系统会有不同的设置，大多数Linux发布版中都是默认线程堆大小为8MB。Apache的架构在大负载下变得不可预测，为每个打开的连接维护一个大的线程池等待数据极易迅速耗光服务器的内存资源。

大多数社交网络应用都会展示实时更新来提醒新消息、状态变化以及用户通知，这就要求客户端需要保持一个打开的连接来等待服务器端的任何响应。这些长连接或推送请求使得Apache的最大线程池迅速饱和。一旦线程池的资源耗尽，服务器将不能再响应新的请求。

异步服务器在这一场景中的应用相对较新，但他们正是被设计用来减轻基于线程的服务器的限制的。当负载增加时，诸如Node.js，lighttpd和Tornodo这样的服务器使用协作的多任务的方式进行优雅的扩展。也就是说，如果当前请求正在等待来自其他资源的数据（比如数据库查询或HTTP请求）时，一个异步服务器可以明确地控制以挂起请求。异步服务器用来恢复暂停的操作的一个常见模式是当合适的数据准备好时调用回调函数。
