---
title: 解决python3.7 + celery4.2.1运行时找不到async
date: 2019-01-09 20:41:54
tags: celery python3.7 celery
categories: python
---

<!--more-->

参考文档：

- <https://github.com/celery/celery/issues/4500>

最近在使用python3.7去运行celery4.2.1时，发现会报以下错误：  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190109203309807.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDE1NjQ4Nw==,size_16,color_FFFFFF,t_70)

原因是python3.7中`async`已经变成了关键字。因此出现这个错误时，需要将报错文件中所有的`async`改为`asynchronous`，并编写如下shell脚本运行：

```shell
TARGET=/Library/Frameworks/Python.framework/Versions/3.7/lib/python3.7/site-packages/celery/backends
cd $TARGET
if [ -e async.py ]
then
    mv async.py asynchronous.py
    sed -n 's/async/asynchronous/g' redis.py
    sed -n 's/async/asynchronous/g' rpc.py
fi
```

运行后，你会发现celery可以正常使用了