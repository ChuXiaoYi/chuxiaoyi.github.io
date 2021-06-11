---
title: fastapi教程-进阶八（Handling Errors）
date: 2020-09-09 15:05:44
tags: python fastapi
categories: fastapi
---


**参考内容**：

- <https://fastapi.tiangolo.com/>

在很多情况下，在服务端出现错误时，我们需要告诉客户端出现了什么错误，例如：

- 客户端没有足够的权限进行该操作。
- 客户端无权访问该资源。
- 客户端尝试访问的项目不存在。

<!--more-->

这时，我们需要返回给客户端400-499范围内的HTTP状态码。接下来介绍如何通过fastapi对服务端错误进行处理并返回给客户端HTTP状态码

## HTTPException

如果希望将错误返回给客户端，可以使用`HTTPException`:

```python
from fastapi import FastAPI, HTTPException

app = FastAPI()

items = {"foo": "The Foo Wrestlers"}


@app.get("/items/{item_id}")
async def read_item(item_id: str):
    if item_id not in items:
        raise HTTPException(status_code=404, detail="Item not found")
    return {"item": items[item_id]}
```

上面这个代码的含义是希望当查询的item\_id不存在时，返回404状态码，并告诉客户端查询的id不存在。

我们先尝试请求`http://127.0.0.1:8000/items/a`，看看服务端会响应什么:  
![[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-3i6smyYE-1599634775682)(evernotecid://FBE381A3-17C7-41D9-AA37-9C5F29FAB396/appyinxiangcom/20545635/ENResource/p221)]](https://img-blog.csdnimg.cn/2020090915000151.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDE1NjQ4Nw==,size_16,color_FFFFFF,t_70#pic_center)

回到代码中，我们注意到这里的`HTTPException`使用了`raise`，而不是`return`，这是为什么呢？

因为`HTTPException`本身是python的异常类，异常我们通常都是需要将它抛出，所以用了`raise`。这也就意味着，当我们触发了`HTTPException`这个异常，余下的代码将不会再执行，并且服务端会直接将错误通过`HTTPException`响应给客户端。

另外，当我们抛出`HTTPException`时，服务端会返回给我们:

```json
{
  "detail": "Item not found"
}
```

如果我们想返回更具体的错误信息，可以对`detail`进行修改，它不仅可以接收字符串，也可以是json、dict、list等结构，例如我们修改代码:

```python
from fastapi import FastAPI, HTTPException

app = FastAPI()

items = {"foo": "The Foo Wrestlers"}


@app.get("/items/{item_id}")
async def read_item(item_id: str):
    if item_id not in items:
        raise HTTPException(status_code=404, detail={'msg': "Item not found"})
    return {"item": items[item_id]}
```

再次请求`http://127.0.0.1:8000/items/a`，此时的响应就发生了变化：

```json
{
  "detail": {
    "msg": "Item not found"
  }
}
```

除此之外，`HTTPException`还可以添加响应头：

```python
from fastapi import FastAPI, HTTPException

app = FastAPI()

items = {"foo": "The Foo Wrestlers"}


@app.get("/items-header/{item_id}")
async def read_item_header(item_id: str):
    if item_id not in items:
        raise HTTPException(
            status_code=404,
            detail="Item not found",
            headers={"X-Error": "There goes my error"},
        )
    return {"item": items[item_id]}
```

再次请求`http://127.0.0.1:8000/items/a`，此时的响应头增加了`X-Error`：

```
content-length: 27
content-type: application/json
date: Tue, 08 Sep 2020 09:15:21 GMT
server: uvicorn
x-error: There goes my error
```

## exception handlers

上面我们介绍了如何在一个接口中，抛出错误信息给客户端，如果此时有多个接口都抛出了异常响应，并且我们希望可以对全局的错误异常响应做统一的处理呢。这时候我们需要用到`@app.exception_handler()`这个装饰器:

```python
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse


class UnicornException(Exception):
    def __init__(self, name: str):
        self.name = name


app = FastAPI()


@app.exception_handler(UnicornException)
async def unicorn_exception_handler(request: Request, exc: UnicornException):
    return JSONResponse(
        status_code=418,
        content={"message": f"Oops! {exc.name} did something. There goes a rainbow..."},
    )


@app.get("/unicorns/{name}")
async def read_unicorn(name: str):
    if name == "yolo":
        raise UnicornException(name=name)
    return {"unicorn_name": name}
```

尝试请求`http://127.0.0.1:8000/unicorns/yolo`，会响应:

```json
{
  "message": "Oops! yolo did something. There goes a rainbow..."
}
```

上面的例子自定义了异常类`UnicornException`，当请求的name=yolo时，会抛出`UnicornException`。但是并不会立刻响应给客户端，而是先经过`unicorn_exception_handler`处理成统一的格式，最后才会响应给客户端，也就是我们上面看到的响应。

## 重写默认的exception handlers

fastapi有很多默认的异常处理器。这些处理器负责在引发HTTPException以及请求中包含无效数据时返回默认的JSON响应。

我们可以使用自己的方法重写这些异常处理器，下面我们来重写`RequestValidationError`和`HTTPException`的异常处理器:

```python
from fastapi import FastAPI, HTTPException
from fastapi.exceptions import RequestValidationError
from fastapi.responses import PlainTextResponse
from starlette.exceptions import HTTPException as StarletteHTTPException

app = FastAPI()


@app.exception_handler(StarletteHTTPException)
async def http_exception_handler(request, exc):
    return PlainTextResponse(str(exc.detail), status_code=exc.status_code)


@app.exception_handler(RequestValidationError)
async def validation_exception_handler(request, exc):
    return PlainTextResponse(str(exc), status_code=400)


@app.get("/items/{item_id}")
async def read_item(item_id: int):
    if item_id == 3:
        raise HTTPException(status_code=418, detail="Nope! I don't like 3.")
    return {"item_id": item_id}
```

当请求的参数是非法参数时，fastapi会主动抛出`RequestValidationError`，此时会通过`validation_exception_handler`处理这个异常，处理的结果是响应文本格式的结果，并设置状态码为400，我们尝试请求`http://127.0.0.1:8000/items/foo`，此时会返回：  
![[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-LmkcvBcZ-1599634775699)(evernotecid://FBE381A3-17C7-41D9-AA37-9C5F29FAB396/appyinxiangcom/20545635/ENResource/p223)]](https://img-blog.csdnimg.cn/20200909150431329.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDE1NjQ4Nw==,size_16,color_FFFFFF,t_70#pic_center)

当请求参数item\_id=3时，会触发`HTTPException`，并通过`http_exception_handler`响应文本格式的信息，并设置状态码为418，尝试请求`http://127.0.0.1:8000/items/3`，此时会返回：  
![[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-tXVpVvNu-1599634775703)(evernotecid://FBE381A3-17C7-41D9-AA37-9C5F29FAB396/appyinxiangcom/20545635/ENResource/p225)]](https://img-blog.csdnimg.cn/20200909150446458.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDE1NjQ4Nw==,size_16,color_FFFFFF,t_70#pic_center)

#### 如何利用`RequestValidationError`结构体

当我们希望可以知道出现错误的请求体以便可以记日志或debug时，我们可以使用`RequestValidationError`的`body`属性：

```python
from fastapi import FastAPI, Request, status
from fastapi.encoders import jsonable_encoder
from fastapi.exceptions import RequestValidationError
from fastapi.responses import JSONResponse
from pydantic import BaseModel

app = FastAPI()


@app.exception_handler(RequestValidationError)
async def validation_exception_handler(request: Request, exc: RequestValidationError):
    return JSONResponse(
        status_code=status.HTTP_422_UNPROCESSABLE_ENTITY,
        content=jsonable_encoder({"detail": exc.errors(), "body": exc.body}),
    )


class Item(BaseModel):
    title: str
    size: int


@app.post("/items/")
async def create_item(item: Item):
    return item
```

尝试向`http://127.0.0.1:8000/items/`发送一个非法的请求体：

```json
{
  "title": "string",
  "size": "aaa"
}
```

我们来看看会响应什么：

```json
{
  "detail": [
    {
      "loc": [
        "body",
        "size"
      ],
      "msg": "value is not a valid integer",
      "type": "type_error.integer"
    }
  ],
  "body": {
    "title": "string",
    "size": "aaa"
  }
}
```

#### FastAPI’s HTTPException vs Starlette’s HTTPException

通过上面的例子我们可以注意到有的例子用了`from fastapi import HTTPException`也有用`from starlette.exceptions import HTTPException as StarletteHTTPException`，那么这两种用法有什么区别呢？

![[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-2XIDY2Nb-1599634775711)(evernotecid://FBE381A3-17C7-41D9-AA37-9C5F29FAB396/appyinxiangcom/20545635/ENResource/p226)]](https://img-blog.csdnimg.cn/20200909150759733.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDE1NjQ4Nw==,size_16,color_FFFFFF,t_70#pic_center)

上面的源码告诉了我们，虽然FastAPI的HTTPException继承自Starlette的HTTPException，但是FastAPI的HTTPException允许添加响应头。  
一般情况下，我们可以使用`from fastapi import HTTPException`，但是当我们去重写`HTTPException`处理器的时候，建议还是使用`from starlette.exceptions import HTTPException as StarletteHTTPException`。原因是，如果是starlette内部代码或相关插件引发了HTTPException，我们可以通过处理器捕捉到它。

## 重用fastapi的exception handlers

如果我们想用默认的异常处理器，但是希望在这之前加一些自己的东西，比如，在默认处理器之前打印一下错误信息：

```python
from fastapi import FastAPI, HTTPException
from fastapi.exception_handlers import (
    http_exception_handler,
    request_validation_exception_handler,
)
from fastapi.exceptions import RequestValidationError
from starlette.exceptions import HTTPException as StarletteHTTPException

app = FastAPI()


@app.exception_handler(StarletteHTTPException)
async def custom_http_exception_handler(request, exc):
    print(f"OMG! An HTTP error!: {repr(exc)}")
    return await http_exception_handler(request, exc)


@app.exception_handler(RequestValidationError)
async def validation_exception_handler(request, exc):
    print(f"OMG! The client sent invalid data!: {exc}")
    return await request_validation_exception_handler(request, exc)


@app.get("/items/{item_id}")
async def read_item(item_id: int):
    if item_id == 3:
        raise HTTPException(status_code=418, detail="Nope! I don't like 3.")
    return {"item_id": item_id}
```

## 总结

- 返回`HTTPException`时，要用`raise`，不要用`return`
- 我们可以自定义异常响应处理器，使用`@app.exception_handler()`来装饰处理器方法
- 我们也可以通过重写fastapi已有的异常响应处理器来达到我们的目的
- 注意FastAPI的HTTPException继承自Starlette的HTTPException的区别，防止出错的话，直接使用`from starlette.exceptions import HTTPException as StarletteHTTPException`

> 上述栗子均放到git上啦，地址：[戳这里](https://github.com/ChuXiaoYi/fastapi)
