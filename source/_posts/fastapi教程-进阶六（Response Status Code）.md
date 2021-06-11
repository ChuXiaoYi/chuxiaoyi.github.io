---
title: fastapi教程-进阶六（Response Status Code）
date: 2020-09-03 14:19:03
tags: python fastapi
categories: fastapi
---


**参考内容**：

- <https://fastapi.tiangolo.com/>

在[fastapi教程-进阶五（Response Model）](https://chuxiaoyi.blog.csdn.net/article/details/108336769)中我们学习了如何控制响应体结构，这节来学习如何使用http状态码：

<!--more-->

```python
from fastapi import FastAPI

app = FastAPI()


@app.post("/items/", status_code=201)
async def create_item(name: str):
    return {"name": name}
```

启动服务，打开`http://127.0.0.1:8000/docs`，我们来看看response：  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200903141853609.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDE1NjQ4Nw==,size_16,color_FFFFFF,t_70#pic_center)

这个例子通过`status_code`来自定义了状态码，201代表了新建资源，但是假如我们在使用状态码时忘记了对应的数字怎么办呢？这里fastapi就很贴心了，它为我们提供了枚举值:  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200903141642786.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDE1NjQ4Nw==,size_16,color_FFFFFF,t_70#pic_center)

> 这里也可以使用`from starlette import status`。

#### HTTP状态码

- 100以上表示“信息”。我们很少直接使用它们。具有这些状态码的响应不能带有主体。
- 200及以上表示“成功”响应。这些是最常使用的。
  - 默认状态代码为200，表示一切正常。
  - 另一个示例是201，“已创建”。通常在数据库中创建新记录后使用。
  - 特殊情况是204，“无内容”。当没有内容返回给客户端时使用此响应，因此该响应必须没有正文。
- 300及更高版本用于“重定向”。具有这些状态代码的响应可能带有或可能没有主体，但304，“未修改”除外，该主体不能具有一个主体。
- 400及更高版本适用于“客户端错误”响应。这些可能最常使用的第二种类型。
  - 对于“未找到”响应，示例为404。
  - 对于来自客户端的一般错误，您可以仅使用400。
- 500及以上是服务器错误。我们几乎永远不会直接使用它们。当您的应用程序代码或服务器中的某些部分出现问题时，它将自动返回这些状态代码之一。

> 上述栗子均放到git上啦，地址：[戳这里](https://github.com/ChuXiaoYi/fastapi)
