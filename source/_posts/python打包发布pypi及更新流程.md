---
title: python打包发布pypi及更新流程
date: 2020-08-05 11:46:39
tags: python 打包 pypi
categories: python
---


## 参考文档

1.  [Python 快速打包发布软件PyPi上](http://yangfangs.github.io/2018/08/06/python-distribution-packages/#%E5%9C%A8pypi%E5%AE%98%E7%BD%91%E6%B3%A8%E5%86%8C%E4%B8%80%E4%B8%AApypi%E4%B8%AA%E4%BA%BA%E8%B4%A6%E6%88%B7%E5%A6%82%E4%B8%8B)
2.  [包含setup.py的非Python文件](https://www.it-swarm.dev/zh/python/%E5%8C%85%E5%90%ABsetuppy%E7%9A%84%E9%9D%9Epython%E6%96%87%E4%BB%B6/968888625/)
3.  [五步法更新pypi包体](https://blog.csdn.net/weixin_41855010/article/details/105506343)

## 发布

<!--more-->

### 1\. 安装打包依赖工具

```python
pip install setuptools
```

### 2\. 安装上传工具

```python
pip install twine
```

### 3\. 注册[PYPI官网](https://pypi.org/)个人用户

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200805113342473.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDE1NjQ4Nw==,size_16,color_FFFFFF,t_70)

### 4\. 在和项目同级目录创建setup.py

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200805113414940.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDE1NjQ4Nw==,size_16,color_FFFFFF,t_70)  
代码如下：

```python
from setuptools import setup, find_packages

GFICLEE_VERSION = '2020.8.4.6'

setup(
    name='cfastproject',
    version=GFICLEE_VERSION,
    packages=find_packages(),
    include_package_data=True,
    entry_points={
        "console_scripts": ['cfastproject = fastproject.main:main']
    },
    install_requires=[
        "django", "fastapi", "gcp_mixed_logging", "asgi_request_id",
        "uvicorn", "google-cloud-secret-manager", "pandas",
        "peewee_async", "aiopg", "aiohttp"
    ],
    url='https://github.com/ChuXiaoYi/fastproject',
    license='GNU General Public License v3.0',
    author='Xiaoyi Chu',
    author_email='895706056@qq.com',
    description='More convenient to create fastapi project'
)
```

setup参数说明：

| 名称 | 描述 | 说明 |
| --- | --- | --- |
| name | 项目名称 | 不可重复 |
| version | 项目版本 | 保证每次发布都是版本都是唯一的 |
| packages | 项目本身的代码 |  |
| include\_package\_data | 是否包括非包文件 |  |
| entry\_points | 项目主入口 | 安装成功后，在命令行输入`cfastproject` 就相当于执行了`fastproject.main.py`中的`main()`了 |
| install\_requires | 项目依赖包 |  |
| url | 项目地址 |  |
| license | license |  |
| author | 项目作者 |  |
| author\_email | 项目邮箱 |  |
| description | 项目描述 |  |

### 5\. 打包前检查

> 通过这一步可以检查setup.py中是否有错误，例如版本号错误

```python
python setup.py check
```

### 6\. 打包

```python
python setup.py sdist
```

### 7\. 发布前准备

1.  在home目录下创建.pypirc 文件，写入pypi账户密码，这样每次上传就不需要在重复输入了

    ```python
    [distutils]
    index-servers =
    pypi

    [pypi]
    username:username
    password:password
    ```

2.  本地测试

    ```python
    python setup.py install
    ```

    安装成功后，可以通过上面定义的命令执行一次，如果成功证明安装成功，可以继续打包了

### 8\. 注册

> 上传前需要注册一下包的名称，因为这个名称必须独一无二，如被占用则注册不通过。

```python
python setup.py register
```

### 9\. 检查是否符合pypi要求

```python
twine check dist/**_.tar.gz
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200805114338643.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDE1NjQ4Nw==,size_16,color_FFFFFF,t_70)

### 10\. 上传

```python
twine upload dist/**_.tar.gz
```

上传成功后，到官网上搜索看看包有木有吧～  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200805114403402.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDE1NjQ4Nw==,size_16,color_FFFFFF,t_70)

## 更新

### 1\. 更新代码，并修改setup.py中的版本号

### 2\. 更新包

```python
python setup.py sdist bdist
```

### 3\. 上传

```python
twine upload dist/**_.tar.gz
```

### 4\. 更新包

```python
pip install --upgrade cfastproject
```

## 关于上传非包文件

在`setup.py`同级目录下创建`MANIFEST.in`文件，里面的内容是需要上传的文件，例如，如果要包括项目下的所有文件：

```python
recursive-include fastproject *
```

> 为了将这些文件在安装时复制到site-packages中的包文件夹，需要将setup中的include\_package\_data设置为True
