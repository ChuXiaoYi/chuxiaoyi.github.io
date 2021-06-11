---
title: fastapi教程-进阶五（Response Model）
date: 2020-09-02 14:47:40
tags: fastapi python
categories: fastapi
---



**参考内容**：

- <https://fastapi.tiangolo.com/>

## Response Model

还是以一个例子开头：

<!--more-->

```python
from typing import Optional

from fastapi import FastAPI
from pydantic import BaseModel, EmailStr

app = FastAPI()


class UserIn(BaseModel):
    username: str
    password: str
    email: EmailStr
    full_name: Optional[str] = None


# Don't do this in production!
@app.post("/user/", response_model=UserIn)
async def create_user(user: UserIn):
    return user
```

我们可以在下面这几个操作中用`response_model`参数来声明响应的模板结构:

- `@app.get()`
- `@app.post()`
- `@app.put()`
- `@app.delete()`
- 其他请求类型

FastAPI将使用`response_model`进行以下操作：

- 将输出数据转换为response\_model中声明的数据类型。
- 验证数据结构和类型
- 将输出数据限制为该model定义的
- 添加到OpenAPI中
- 在自动文档系统中使用。

我们尝试启动上面这个例子的服务，并通过POST请求`http://127.0.0.1:8000/user/`，我们可以看到返回和请求体是一样的:  
![[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-1sVQyk8E-1599029098971)(evernotecid://FBE381A3-17C7-41D9-AA37-9C5F29FAB396/appyinxiangcom/20545635/ENResource/p216)]](https://img-blog.csdnimg.cn/2020090214460145.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDE1NjQ4Nw==,size_16,color_FFFFFF,t_70#pic_center)

不难发现这里有一个问题，如果我们用同一个model去声明请求体参数和响应model，会造成password的泄露，那么，我们来尝试不让password返回：

```python
from typing import Optional

from fastapi import FastAPI
from pydantic import BaseModel, EmailStr

app = FastAPI()


class UserIn(BaseModel):
    username: str
    password: str
    email: EmailStr
    full_name: Optional[str] = None


class UserOut(BaseModel):
    username: str
    email: EmailStr
    full_name: Optional[str] = None


@app.post("/user/", response_model=UserOut)
async def create_user(user: UserIn):
    return user
```

这里我们定义了两个model（UserIn和UserOut）分别用在请求体参数和响应model，再次请求`http://127.0.0.1:8000/user/`，这时我们发现响应已经没有password了：

```json
{
  "username": "string",
  "email": "user@example.com",
  "full_name": "string"
}
```

#### response\_model\_exclude\_unset

通过上面的例子，我们学到了如何用response\_model控制响应体结构，但是如果它们实际上没有存储，则可能要从结果中忽略它们。

例如，如果model在NoSQL数据库中具有很多可选属性，但是不想发送很长的JSON响应，其中包含默认值。

我们来看一个例子：

```python
from typing import List, Optional

from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    description: Optional[str] = None
    price: float
    tax: float = 10.5
    tags: List[str] = []


items = {
    "foo": {"name": "Foo", "price": 50.2},
    "bar": {"name": "Bar", "description": "The bartenders", "price": 62, "tax": 20.2},
    "baz": {"name": "Baz", "description": None, "price": 50.2, "tax": 10.5, "tags": []},
}


@app.get("/items/{item_id}", response_model=Item, response_model_exclude_unset=True)
async def read_item(item_id: str):
    return items[item_id]
```

我们发现，get操作中多了`response_model_exclude_unset`属性，他是用来控制什么的呢？

尝试请求`http://127.0.0.1:8000/items/foo`，结果返回了这样的数据：

```json
{
  "name": "Foo",
  "price": 50.2
}
```

通过前面的学习，我们知道response的结构应该是这样的：

```json
{
  "name": "string",
  "description": "string",
  "price": 0,
  "tax": 0,
  "tags": [
    "string"
  ]
}
```

这就是`response_model_exclude_unset`发挥了作用。`response_model_exclude_unset`可以控制不返回没有设置的参数。  
当然，除了`response_model_exclude_unset`以外，还有`response_model_exclude_defaults`和`response_model_exclude_none`，我们可以很直观的了解到他们的意思，不返回是默认值的字段和不返回是None的字段，下面让我们分别尝试一下

设置`response_model_exclude_defaults=True`, 请求`http://127.0.0.1:8000/items/baz`，返回：

```json
{
  "name": "Baz",
  "price": 50.2
}
```

设置`response_model_exclude_none=True`, 请求`http://127.0.0.1:8000/items/baz`，返回：

```json
{
  "name": "Baz",
  "price": 50.2,
  "tax": 10.5,
  "tags": []
}
```

> 注意：`response_model_exclude_defaults`控制了不返回是默认值的字段，这个默认值不仅仅是None，他可能是我们在model中设置的任意一个值

#### response\_model\_include和response\_model\_exclude

接下来我们学习如何用`response_model_include`和`response_model_exclude`来控制响应参数，不难理解这两个参数的意思分别是响应参数包括哪些和不包括哪些，我们还是从实例看看他们是如何使用的：

```python
from typing import Optional

from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    description: Optional[str] = None
    price: float
    tax: float = 10.5


items = {
    "foo": {"name": "Foo", "price": 50.2},
    "bar": {"name": "Bar", "description": "The Bar fighters", "price": 62, "tax": 20.2},
    "baz": {
        "name": "Baz",
        "description": "There goes my baz",
        "price": 50.2,
        "tax": 10.5,
    },
}


@app.get(
    "/items/{item_id}/name",
    response_model=Item,
    response_model_include={"name", "description"},
)
async def read_item_name(item_id: str):
    return items[item_id]


@app.get("/items/{item_id}/public", response_model=Item, response_model_exclude={"tax"})
async def read_item_public_data(item_id: str):
    return items[item_id]
```

通过前面的学习，我们知道了这里响应的结构应该是：

```json
{
  "name": "string",
  "description": "string",
  "price": 0,
  "tax": 0
}
```

那么，我们来尝试一下请求`http://127.0.0.1:8000/items/foo/name`, 返回：

```json
{
  "name": "Foo",
  "description": null
}
```

为什么只返回了`name`和`description`呢？因为我们在参数中添加了`response_model_include={"name", "description"}`，它控制了响应参数只包括`name`和`description`。  
再次尝试请求`http://127.0.0.1:8000/items/bar/public`，返回：

```json
{
  "name": "Bar",
  "description": "The Bar fighters",
  "price": 62
}
```

为什么没有返回`tax`呢？这时因为我们在参数中添加了`response_model_exclude={"tax"}`，它控制了想响应参数除了`tax`都要返回。

> 除了用`set`来声明`response_model_exclude`和`response_model_include`，也可以用`list`或`tuple`，fastapi会将他们专程set类型

## 总结

1.  使用路径操作装饰器的参数response\_model定义响应模型，确保私有数据被过滤掉
2.  使用路径操作装饰器的参数`response_model_exclude_unset`、`response_model_exclude_defaults`和`response_model_exclude_none`来过滤掉符合条件的参数
3.  使用路径操作装饰器的参数`response_model_include`和`response_model_exclude`来指定字段进行过滤

> 上述栗子均放到git上啦，地址：[戳这里](https://github.com/ChuXiaoYi/fastapi)
