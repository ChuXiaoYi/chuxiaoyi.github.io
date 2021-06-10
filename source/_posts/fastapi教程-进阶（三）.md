---
title: fastapi教程-进阶（三）
date: 2020-08-28 18:11:06
tags: python python fastapi
categories: fastapi
---

<!--more-->

**参考内容**：

- <https://fastapi.tiangolo.com/>

在fastapi教程的前几篇教程里，我们学习了如何声明路径参数、查询参数和请求体，这篇我们会对这些参数进行扩展学习，学习更多的用法

## Query Parameters

我们先看一个例子：

```python
from typing import Optional

from fastapi import FastAPI, Query

app = FastAPI()


@app.get("/items/")
async def read_items(q: Optional[str] = Query(None, min_length=3, max_length=50, regex="^fixedquery$")):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

与之前的例子相比，发生了一些改变：

```python
from fastapi import FastAPI, Query
```

```python
async def read_items(q: Optional[str] = Query(None, min_length=3, max_length=50, regex="^fixedquery$")):
```

声明参数的时候，不再是单纯的指定`None`，而是调用了`Query`。  
通过这种方式，对`q`做了验证限制：

1.  非必填
2.  最大长度为50
3.  最小长度为3
4.  可以匹配正则表达式

尝试请求`http://127.0.0.1:8000/items/?q=a`，会发现返回了错误提示：

```json
{
  "detail": [
    {
      "loc": [
        "query",
        "q"
      ],
      "msg": "ensure this value has at least 3 characters",
      "type": "value_error.any_str.min_length",
      "ctx": {
        "limit_value": 3
      }
    }
  ]
}
```

再次尝试`http://127.0.0.1:8000/items/?q=test`:

```json
{
  "detail": [
    {
      "loc": [
        "query",
        "q"
      ],
      "msg": "string does not match regex \"^fixedquery$\"",
      "type": "value_error.str.regex",
      "ctx": {
        "pattern": "^fixedquery$"
      }
    }
  ]
}
```

除此之外，`Query`还支持其他的属性：

 1.     alias: 参数别名
 2.     description: 参数描述
 3.     title：参数标题
 4.     deprecated：表示该接口已被移除

```python
from typing import Optional

from fastapi import FastAPI, Query

app = FastAPI()


@app.get("/items/")
async def read_items(
        q: Optional[str] = Query(..., title="Query string",
                                 description="Query string for the items to search in the database that have a good match",
                                 alias="item-query")):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

打开`http://127.0.0.1:8000/docs`， 我们会发现页面发生了变化，并且请求的参数名也变成了别名：  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200828170100648.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDE1NjQ4Nw==,size_16,color_FFFFFF,t_70#pic_center)  
这里我们会发现`item-query`变成了必填，这是因为，我们将`Query(None)`替换成了`Query(...)`

## Path Parameters

依旧是一个例子：

```python
from fastapi import FastAPI, Path, Query

app = FastAPI()


@app.get("/items/{item_id}")
async def read_items(
        *,
        item_id: int = Path(..., title="The ID of the item to get", ge=0, le=1000),
        q: str,
        size: float = Query(..., gt=0, lt=10.5)
):
    results = {"item_id": item_id}
    if q:
        results.update({"q": q})
    return results
```

在这里，用`Path`来声明路径参数，通过这种方式，对`item_id`做了验证限制：

1.  `item_id`是必填的
2.  `item_id`大于等于0
3.  `item_id`小于等于1000

**除此之外，我们可以发现方法的第一个参数是`*`，它的作用什么呢？**  
如果我们想不用`Query`来声明`q`，并想将q设为必填的，并且想用`Path`来声明`item_id`，这时我们就要用到`*`。因为python并不会对`*`做任何操作，但是python会将`*`之后的参数作为关键字参数来看待，因此，即使没有用`Query`，python也会把它当作有`Query(...)`来待

> 在从`fastapi`导入`Path`、`Query`时，其实导入的是方法，当它们被调用的时候，会返回同名的类。这些同名的类都是`Param`的子类，因此在声明参数时，他们都是有相同的属性的。![在这里插入图片描述](https://img-blog.csdnimg.cn/20200828181006807.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDE1NjQ4Nw==,size_16,color_FFFFFF,t_70#pic_center)

## Body Parameters

#### 多个body参数

还是以一个例子开始

```python
from typing import Optional

from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    description: Optional[str] = None
    price: float
    tax: Optional[float] = None


class User(BaseModel):
    username: str
    full_name: Optional[str] = None


@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item, user: User):
    results = {"item_id": item_id, "item": item, "user": user}
    return results
```

在这个例子里，fastapi会注意到不止一个body参数，此时的请求体是这样的：

```json
{
  "item": {
    "name": "string",
    "description": "string",
    "price": 0,
    "tax": 0
  },
  "user": {
    "username": "string",
    "full_name": "string"
  }
}
```

fastapi会根据请求自动处理参数，以便用item和user可以接收到其内容  
假如此时我们想在请求体中增加一个参数，例如：

```json
{
    "item": {
        "name": "Foo",
        "description": "The pretender",
        "price": 42.0,
        "tax": 3.2
    },
    "user": {
        "username": "dave",
        "full_name": "Dave Grohl"
    },
    "importance": 5
}
```

这时我们该如何写参数呢

```python
from typing import Optional

from fastapi import Body, FastAPI
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    description: Optional[str] = None
    price: float
    tax: Optional[float] = None


class User(BaseModel):
    username: str
    full_name: Optional[str] = None


@app.put("/items/{item_id}")
async def update_item(
    item_id: int, item: Item, user: User, importance: int = Body(...)
):
    results = {"item_id": item_id, "item": item, "user": user, "importance": importance}
    return results
```

#### 嵌入单一body参数

一般情况下，我们声明一个body参数时，fastapi会这样解析：

```json
{
  "name": "string",
  "description": "string",
  "price": 0,
  "tax": 0
}
```

如果我们想要下面这样的结构呢

```json
{
  "item": {
    "name": "string",
    "description": "string",
    "price": 0,
    "tax": 0
  }
}
```

可以这样写

```python
from typing import Optional

from fastapi import Body, FastAPI
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    description: Optional[str] = None
    price: float
    tax: Optional[float] = None


@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item = Body(..., embed=True)):
    results = {"item_id": item_id, "item": item}
    return results
```

#### Field

前面我们学习了如果给`Query`、`Path`增加验证限制，这里会介绍`Body`如何增加验证限制

```python
from typing import Optional

from fastapi import Body, FastAPI
from pydantic import BaseModel, Field

app = FastAPI()


class Item(BaseModel):
    name: str
    description: Optional[str] = Field(
        None, title="The description of the item", max_length=300
    )
    price: float = Field(..., gt=0, description="The price must be greater than zero")
    tax: Optional[float] = None


@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item = Body(..., embed=True)):
    results = {"item_id": item_id, "item": item}
    return results
```

> 通过上面的例子不难看出`Field`的属性和`Query`、`Path`是一样的。上面我们说过`Query`、`Path`是`Param`的子类，这里的`Field`本身是方法，返回了`FieldInfo`类，而`Param`是`FieldInfo`的子类，因此他们的属性都是一样的![在这里插入图片描述](https://img-blog.csdnimg.cn/20200831142739112.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDE1NjQ4Nw==,size_16,color_FFFFFF,t_70#pic_center)  
> `Body`也是一个方法，他返回了继承自`FieldInfo`的`Body`类![在这里插入图片描述](https://img-blog.csdnimg.cn/20200831143036952.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDE1NjQ4Nw==,size_16,color_FFFFFF,t_70#pic_center)

> 上述栗子均放到git上啦，地址：[戳这里](https://github.com/ChuXiaoYi/fastapi)