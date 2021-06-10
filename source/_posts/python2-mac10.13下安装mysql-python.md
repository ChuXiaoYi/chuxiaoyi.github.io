---
title: python2-mac10.13下安装mysql-python
date: 2018-12-28 13:24:17
tags: myql python2.7 mysql
categories: python
---

<!--more-->

安装环境：OS X 操作系统，Python 2.7.3。

MySQLdb 其实包含在 MySQL-python 包中，因此无论下载还是在 pip 中 search，都应该是搜寻 MySQL-python。

以下为安装步骤

#### 安装MYSQLdb

在 SourceForge 可以下载 [MySQL-python-1.2.4b4.tar](https://sourceforge.net/projects/mysql-python/)，下载后解压，然后在终端 Terminal 中执行以下命令：

```python
(venv) ➜  MySQL-python-1.2.4b4    python setup.py install
```

但是，可能会出现这样的问题：  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181228132213220.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDE1NjQ4Nw==,size_16,color_FFFFFF,t_70)

不要慌，接下来在终端中输入：

```python
(venv) ➜  MySQL-python-1.2.4b4 wget https://pypi.python.org/packages/source/d/distribute/distribute-0.6.28.tar.gz
```

接下来，继续安装，会发现可以安装了！

```python
(venv) ➜  MySQL-python-1.2.4b4 sudo python setup.py install
```

但是你会发现，又出现了另外一个错误  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181228132241849.png)

此时，在终端输入

```python
(venv) ➜  MySQL-python-1.2.4b4 brew install mysql-connector-c 
```

然后继续执行

```python
(venv) ➜  MySQL-python-1.2.4b4 sudo python setup.py install
```

这时，你会发现安装成功了！  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181228132304145.png)  
最后安装MYSQL-python

```python
pip2 install MYSQL-python
```

打开python进行验证，发现又遇到了另一个问题  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181228132318540.png)  
在终端输入：

```python
sudo ln -s /usr/local/mysql/lib/libmysqlclient.21.dylib /usr/local/lib/libmysqlclient.21.dylib
```

但是会发现有权限问题，解决方法是：

```
重启电脑，开机时按住 cmd + R，进入 Recovery 模式。然后打开终端工具 ，输入命令：csrutil diable，然后再次重启电脑即可。
```

然后继续在终端输入：

```python
(venv) ➜  lib sudo ln -s /usr/local/mysql/lib/libmysqlclient.21.dylib /usr/local/lib/libmysqlclient.21.dylib
(venv) ➜  lib sudo ln -s /usr/local/mysql/lib/libmysqlclient.21.dylib /usr/lib/libmysqlclient.21.dylib
```

好啦。现在再次检查，成功！  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181228132336290.png)