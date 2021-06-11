---
title: fastapi教程-进阶四（cookie、header parameter）
date: 2020-09-01 11:02:36
tags: python fastapi
categories: fastapi
---


**参考内容**：

- <https://fastapi.tiangolo.com/>

在[fastapi教程-进阶（三）](https://blog.csdn.net/weixin_40156487/article/details/108281234)和[fastapi教程-进阶（二）](https://blog.csdn.net/weixin_40156487/article/details/108279120)中我们介绍了`Query`、`Path`和`Body`参数，这里介绍`cookie`和`header`

<!--more-->

## Cookie

```python
from typing import Optional

from fastapi import Cookie, FastAPI

app = FastAPI()


@app.get("/items/")
async def read_items(ads_id: Optional[str] = Cookie(None)):
    return {"ads_id": ads_id}
```

> 要获取cookie，必须需要使用`Cookie`来声明，否则参数将被解释为查询参数。

## Header

```python
from typing import Optional

from fastapi import FastAPI, Header

app = FastAPI()


@app.get("/items/")
async def read_items(user_agent: Optional[str] = Header(None)):
    return {"User-Agent": user_agent}
```

启动服务，并尝试请求`http://127.0.0.1:8000/items/`，会返回：

```json
{"User-Agent":"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/84.0.4147.135 Safari/537.36"}
```

**我们会发现，代码中的`user_agent`会自动获取请求头中的`User-Agent`的值，但是他们的大小写和符号并不相同，这是为什么呢？**

> 大部分请求头中的key是用`-`来分割的，比如`User-Agent`，但是这种命名在python中是不符合规范的，因此，`Header`会自动将参数名称中的下划线`_`转换为连字符`-`。另外，http请求头不区分大小写，因此我们可以用符合python规范的命名方法来表示他们。

如果由于某种原因需要禁用下划线`_`到连字符`-`的自动转换，需要将Header的参数convert\_underscores设置为False：

```python
from typing import Optional

from fastapi import FastAPI, Header

app = FastAPI()


@app.get("/items/")
async def read_items(
        user_agent: Optional[str] = Header(None, convert_underscores=False)
):
    return {"User-Agent": user_agent}
```

这时我们在请求`http://127.0.0.1:8000/items/`时，会返回：

```json
{"User-Agent":null}
```

#### 重复的请求头

如果请求头中同一个key有多个value，例如：

```
X-Token: foo
X-Token: bar
```

这时候应该如何定义呢？

```python
from typing import List, Optional

from fastapi import FastAPI, Header

app = FastAPI()


@app.get("/items/")
async def read_items(x_token: Optional[List[str]] = Header(None)):
    return {"X-Token values": x_token}
```

我们用postman模拟请求  
![[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-ZcD2UYHg-1598929224418)(evernotecid://FBE381A3-17C7-41D9-AA37-9C5F29FAB396/appyinxiangcom/20545635/ENResource/p215)]](https://img-blog.csdnimg.cn/20200901110138328.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDE1NjQ4Nw==,size_16,color_FFFFFF,t_70#pic_center)

会返回：

```json
{
    "User-Agent": null,
    "X-Token values": [
        "foo",
        "bar"
    ]
}
```

## 总结

1.  如果要获取cookie或header，必须要用`Cookie`或`Header`来声明参数，否则fastapi会把参数当作查询参数
2.  `Header`会自动转换请求头中的参数，将参数名称中的下划线`_`转换为连字符`-`，因此在参数命名时我们可以遵守python规范
3.  如果不想让`Header`自动转换可以给`Header`设置`convert_underscores=False`
4.  获取重复请求头时只需要用`List`来声明参数类型

> 上述栗子均放到git上啦，地址：[戳这里](https://github.com/ChuXiaoYi/fastapi)
