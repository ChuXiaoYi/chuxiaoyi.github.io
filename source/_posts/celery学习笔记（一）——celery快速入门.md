---
title: celery学习笔记（一）——celery快速入门
date: 2018-11-07 11:54:58
tags: celery celery
categories: python
---

<!--more-->

**参考文档：**

- <https://www.cnblogs.com/forward-wang/p/5970806.html>

- <http://www.cnblogs.com/shizhengwen/p/6911043.html>

**环境：**

- python： 3.7

- celery： 4.2.1

## 什么是生产者与消费者模式

在实际的软件开发过程中，经常会碰到如下场景：某个模块负责产生数据，这些数据由另一个模块来负责处理（此处的模块是广义的，可以是类、函数、线程、进程等）。产生数据的模块，就形象地称为生产者；而处理数据的模块，就称为消费者。

生产者消费者模式是通过一个容器来解决生产者和消费者的强耦合问题。生产者和消费者彼此之间不直接通讯，而通过消息队列（缓冲区）来进行通讯，所以生产者生产完数据之后不用等待消费者处理，直接扔给消息队列，消费者不找生产者要数据，而是直接从消息队列里取，消息队列就相当于一个缓冲区，平衡了生产者和消费者的处理能力。这个消息队列就是用来给生产者和消费者解耦的。

如图所示：

![image_1crk5j4k71nth1hg015vnko8jdd9.png-9.9kB](http://static.zybuluo.com/chuxiaoyi/zxy45sv0m24dmt85rkhdyubt/image_1crk5j4k71nth1hg015vnko8jdd9.png)

通过这样的模式，可以解决并发的问题，而celery就有效的利用这样一个模式的框架。

## 什么是celery

Celery 是一个基于python开发的分布式异步消息任务队列，通过它可以轻松的实现任务的异步处理。

任务队列的输入是称为任务的工作单元，专用工作进程不断监视任务队列以执行新工作。

celery通过消息进行通信，通常使用broker（消息队列）将客户端与worker解藕。客户端向broker中添加消息，然后broker将消息传递给worker。

celery可以有多个worker和broker，这实现了高可用和水平扩展。

如图所示：

![image_1crk6ab8k6b0mmsvvh1u4cve7m.png-15.1kB](http://static.zybuluo.com/chuxiaoyi/mbrw4uzxh97cv8tllr5awxt6/image_1crk6ab8k6b0mmsvvh1u4cve7m.png)

**celery的架构主要由三部分组成：**

- 消息队列（消息中间件）：celery本身无法接收或发送消息，这个时候就需要一个中间人帮celery来完成。这个中间人就是消息中间件，也就是上面说的broker。broker包括：[RabbitMQ](http://www.rabbitmq.com/documentation.html),[Redis](http://www.redis.net.cn/tutorial/3501.html),[MongoDB](https://www.mongodb.com/)等

- worker：执行任务的基本单元。一个celery可以有多个worker。worker运行在分布式的各个系统中。

- backend：用来存储任务的执行结果。celery支持很多种存储结构的方法：Redis，MongoDB，Django ORM，AMQP等

如图所示：

![image_1crk8hjg01a0ibd9mra5a1tel13.png-30.6kB](http://static.zybuluo.com/chuxiaoyi/togusggtlnyqudkyc1a9304l/image_1crk8hjg01a0ibd9mra5a1tel13.png)

## 快速入门

celery的架构分为三部分：broker，worker，backend

那么首先，需要选择一个适合我们的broker。这里会介绍rabbitMQ和redis作为broker的用法

- rabbitMQ

  - 安装

    ```
    brew install rabbitmq
    ```

    安装后的地址在：`/usr/local/Cellar/`

    将`/usr/local/Cellar/rabbitmq/3.7.8/sbin`添加到`.bash_profile`中，便于使用

    ```
    export PATH=$PATH:/usr/local/Cellar/rabbitmq/3.7.8/sbin
    ```

  - 启动与停止

    ```
    sudo rabbitmq-server
    ```

    ![image_1crkbmffqatp134q14hj1qkn11c7m.png-31.6kB](http://static.zybuluo.com/chuxiaoyi/lmj4ijmlbm4tjmher44ili1n/image_1crkbmffqatp134q14hj1qkn11c7m.png)

    注意，rabbimq停止的时候不要用kill命令，要用下面这个：

    ```
    sudo rabbitmqctl stop
    ```

- redis

  - 安装

    ```
    pip3 install -U "celery[redis]"
    ```

    [redis安装包戳这里](http://download.redis.io/releases/)

  - 启动与停止

    ```
    redis-server
    ```

    ![image_1crm0m6qe1jvuc5jnkgldk9as9.png-58kB](http://static.zybuluo.com/chuxiaoyi/8xgdv7f84vusoloaek576a4l/image_1crm0m6qe1jvuc5jnkgldk9as9.png)

    ```
    redis-cli shutdown
    ```

下面，我们来安装celery

```
pip3 install celery
```

实现一个celery架构的第一步，就是需要一个celery实例。通常这个实例叫app，并且这个实例将作为一切你想在celery中做的事的入口。所以，必须保证app所在的module可以让任意一个文件导入。

这里我们做一个简单的示例，把所有东西都写在一个module（[tasks.py](http://tasks.py)）中

```python
from celery import Celery

app = Celery('tasks', broker='pyamqp://guest@localhost//')


@app.task
def add(x, y):
    return x + y
```

Celery的第一个参数“tasks”是当前module的名称。只有当“**name** == ‘**main**’”的时候才能自动生成名称

第二个参数“broker”，指定了使用的消息中间件的url，这里使用的是rabbitmq

使用“\@app.task”装饰的“add”，将作为任务放到任务队列中

运行一下，看看效果

```
celery -A tasks worker --loglevel=info
```

![image_1crm304c81kuj70mjtrr0k1vk62m.png-71.7kB](http://static.zybuluo.com/chuxiaoyi/jsxht7u3za82i3itbsd6zp3h/image_1crm304c81kuj70mjtrr0k1vk62m.png)

celery启动后，我们需要向celery中添加任务

```
>>> from tasks import add
>>> add.delay(1, 1)
<AsyncResult: f17075c0-f36d-44e9-87a0-df7a24d66438>
```

`delay()`是`apply_async()`的一种快捷使用方式，用来更好的控制任务的执行

控制台打印了如下信息：

![image_1crm3ihba5v46cnjrkaag1dhr4j.png-17.4kB](http://static.zybuluo.com/chuxiaoyi/2wu91socwh0hmjn2zbczk0g0/image_1crm3ihba5v46cnjrkaag1dhr4j.png)

我们可以看到，add任务已经被worker处理。

调用一个任务，将返回一个AsyncResult实例。这个实例可以用来检查任务的状态，从而等待任务完成并获取其返回值

这里我们没有设置backend，所以在通过AsyncResult实例检查任务状态时，会报错`AttributeError: 'DisabledBackend' object has no attribute '_get_task_meta_for'`

上面提到的会报错，原因是没有一个backend可以用来存储任务的状态及结果，因此，我们在上面的tasks.py中，添加backend参数

```python
from celery import Celery

app = Celery('tasks', broker='pyamqp://guest@localhost//', backend='redis://localhost')


@app.task
def add(x, y):
    return x + y
```

重新启动celery，再次调用任务

```
>>> from tasks import add
>>> result = add.delay(1, 1)
```

这时我们会发现控制台打印了和上面一样的内容

同时，我们通过result（这是一个AsyncResult实例）检查任务的状态

```
>>> result.ready()
True
```

结果为True，证明已经执行结束，此时，我们查看结果

```
>>> result.get()
2
>>> result.result
2
```

这里我们用了两个方式去获取结果，他们的区别在于`get()`在获取到结果的时候，同时释放了资源，而`result`却没有。backend在存储和传输结果的时候，会占用资源，因此推荐使用`get()`。

我们的backend使用了redis来存储，那么我们去redis中看一看结果

![image_1crm4ercmsnj1of553jucf18bv50.png-35.5kB](http://static.zybuluo.com/chuxiaoyi/ie0e8qfr716s75tpwx45qjnc/image_1crm4ercmsnj1of553jucf18bv50.png)

这样一个celery的简单入门就结束啦～～～