---
title: fastapi教程-进阶（二）
date: 2020-08-28 15:06:45
tags: python python fastapi
categories: fastapi
---

<!--more-->

**参考内容**：

- <https://fastapi.tiangolo.com/>

## Request Body

这里我们来介绍一下POST请求时，fastapi是如何接收请求体的

```python
from typing import Optional

from fastapi import FastAPI
from pydantic import BaseModel


class Item(BaseModel):
    name: str
    description: Optional[str] = None
    price: float
    tax: Optional[float] = None


app = FastAPI()


@app.post("/items/")
async def create_item(item: Item):
    return item
```

接下来，我们来拆解一下步骤：

**1\. 导入`BaseModel`**

```python
from pydantic import BaseModel
```

**2\. 声明请求体结构，这个结构体是一个继承了`BaseModel`的类**

```python
class Item(BaseModel):
    name: str
    description: Optional[str] = None
    price: float
    tax: Optional[float] = None
```

> 如果指定了默认值，那么他就是非必填的；否则，这个参数就是必填的。  
> 这里用None来表示参数为非必填，在传递参数时，可以不写这个参数  
> 根据上面的结构体，我们可以这样传递参数：

```json
{
    "name": "Foo",
    "description": "An optional description",
    "price": 45.2,
    "tax": 3.5
}
```

或者

```json
{
    "name": "Foo",
    "price": 45.2
}
```

**3\. 将声明好的结构体添加到方法中**

```python
@app.post("/items/")
async def create_item(item: Item):
    return item
```

这时，我们的接口已经初步完成，当有请求进来是，fastapi会这样做：

1.  读取json格式的请求体
2.  将参数转换为对应的类型
3.  验证数据是否合法，如果不符合定义的类型会返回错误的字段及错误的原因

运行服务，打开`http://127.0.0.1:8000/docs`，可以看到:  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200828145020619.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDE1NjQ4Nw==,size_16,color_FFFFFF,t_70#pic_center)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200828144920693.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDE1NjQ4Nw==,size_16,color_FFFFFF,t_70#pic_center)

## Request body + path + query parameters

在[fastapi教程——进阶（一）](https://blog.csdn.net/weixin_40156487/article/details/102496066)我们已经学过了如何声明path和query参数，接下来我们把它们整理到一起：

```python
from typing import Optional

from fastapi import FastAPI
from pydantic import BaseModel


class Item(BaseModel):
    name: str
    description: Optional[str] = None
    price: float
    tax: Optional[float] = None


app = FastAPI()


@app.put("/items/{item_id}")
async def create_item(item_id: int, item: Item, q: Optional[str] = None):
    result = {"item_id": item_id, **item.dict()}
    if q:
        result.update({"q": q})
    return result
```

这个接口的方法将会这样获取参数：

1.  如果在路径中声明了该参数，那么他就会被传递给`item_id`
2.  如果在查询参数中指定了参数，那么他就会被传递给`q`
3.  如果在请求体中指定了参数，那么他就会被传递给`item`

上述栗子均放到git上啦，地址：[戳这里](https://github.com/ChuXiaoYi/fastapi)