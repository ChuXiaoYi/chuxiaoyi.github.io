---
title: fastapi——快速入门
date: 2019-09-26 15:27:51
tags: fastapi python fastapi
categories: python
---



> fastapi是高性能的web框架。他的主要特点是：  
> \- 快速编码  
> \- 减少人为bug  
> \- 直观  
> \- 简易  
> \- 具有交互式文档  
> \- 基于API的开放标准（并与之完全兼容）：OpenAPI（以前称为Swagger）和JSON Schema。

技术背景：python3.6+、[Starlette](https://www.starlette.io/)、[Pydantic](https://pydantic-docs.helpmanual.io/)

<!--more-->

## 安装

```
pip install fastapi
pip install uvicorn
```

## quick start

### main.py

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/")
def read_root():
    return {"Hello": "World"}


@app.get("/items/{item_id}")
def read_item(item_id: int, q: str = None):
    return {"item_id": item_id, "q": q}
```

或者

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/")
async def read_root():
    return {"Hello": "World"}


@app.get("/items/{item_id}")
async def read_item(item_id: int, q: str = None):
    return {"item_id": item_id, "q": q}
```

### 运行

```
uvicorn main:app --reload
```

看到如下提示，证明运行成功  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190926152850404.png)

> main: 表示app所在文件名  
> app：FastAPI实例  
> reload：debug模式，可以自动重启

试着请求`http://127.0.0.1:8000/items/5?q=somequery`，会看到如下返回

```
{"item_id": 5, "q": "somequery"}
```

### 交互文档

试着打开`http://127.0.0.1:8000/docs`  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190926153457923.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDE1NjQ4Nw==,size_16,color_FFFFFF,t_70)

### API文档

试着打开http://127.0.0.1:8000/redoc  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190926153624102.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDE1NjQ4Nw==,size_16,color_FFFFFF,t_70)

## update

通过上面的例子，我们已经用fastapi完成了第一个web服务，现在我们再添加一个接口

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    price: float
    is_offer: bool = None


@app.get("/")
def read_root():
    return {"Hello": "World"}


@app.get("/items/{item_id}")
def read_item(item_id: int, q: str = None):
    return {"item_id": item_id, "q": q}


@app.put("/items/{item_id}")
def update_item(item_id: int, item: Item):
    return {"item_name": item.name, "item_id": item_id}
```

此时会发现，服务自动重启了，这是因为我们在启动命令后添加了`--reload`。再次查看文档，发现同样发生了改变。  
到此，你已经可以快速的用fastapi搭建起服务了～
