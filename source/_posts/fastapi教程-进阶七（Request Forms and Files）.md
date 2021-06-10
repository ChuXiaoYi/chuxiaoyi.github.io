---
title: fastapi教程-进阶七（Request Forms and Files）
date: 2020-09-04 15:21:17
tags: python fastapi
categories: fastapi
---

<!--more-->

**参考内容**：

- <https://fastapi.tiangolo.com/>

## Form Data

前面介绍的参数都是以json格式传递的，这节我们来介绍表单参数如何使用

> 如果要使用表单参数要先安装`python-multipart`  
> `pip install python-multipart`

下面这个例子模拟了登陆的表单验证，我们可以看到，参数的声明没有用`Body`或者`Query`，而是用了`Form`

```python
from fastapi import FastAPI, Form

app = FastAPI()


@app.post("/login/")
async def login(username: str = Form(...), password: str = Form(...)):
    return {"username": username}
```

打开`http://127.0.0.1:8000/docs`，我们来看一下接口文档发生了哪些变化：  
![[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-3s9e9Jge-1599203933286)(evernotecid://FBE381A3-17C7-41D9-AA37-9C5F29FAB396/appyinxiangcom/20545635/ENResource/p219)]](https://img-blog.csdnimg.cn/20200904151911191.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDE1NjQ4Nw==,size_16,color_FFFFFF,t_70#pic_center)

很明显的一点就是content-type发生了变化，变成了`Content-Type: application/x-www-form-urlencoded`

> 需要注意的一点是，如果我们同时使用了`Form`和`Body`，请求将会以表单的形式发送给服务端

## Request Files

介绍完表单，接下来介绍文件上传，在使用之前记得安装`python-multipart`

```python
from fastapi import FastAPI, File, UploadFile

app = FastAPI()


@app.post("/files/")
async def create_file(file: bytes = File(...)):
    return {"file_size": len(file)}


@app.post("/uploadfile/")
async def create_upload_file(file: UploadFile = File(...)):
    return {"filename": file.filename}
```

这里有两个接口，可以注意到`file`接收的类型是不同的，那么这两种写法有什么区别呢：

- file是`bytes`类型时，fastapi会读取上传的文件，file参数将接收字节类型的内容，这也意味着全部内容将存储在内存中，如果文件很大的话，这并不是一个好的选择
- file是`UploadFile`类型时，相比`bytes`类型来说，对大文件的处理更好一些，而且可以支持异步

#### UploadFile

接下来我们来仔细了解一下`UploadFile`，为什么它相比`bytes`来说更有优势，更适合大文件：

- `UploadFile`虽然也会读取文件到内存，但是当存储在内存的文件大小达到最大限制后，文件将会存储到硬盘中。这也就意味着它可以很好地用于大型文件，例如图像，视频，大型二进制文件等，而不会占用所有内存。

- 我们通过`UploadFile`可以获取文件的相关属性，例如：

  - filename：上传的文件的文件名（例：myimage.jpg）
  - content\_type：文件类型\(例：image/jpeg\)
  - file：SpooledTemporaryFile（类似文件的对象）。它实际的Python文件，您可以将其直接传递给需要“类文件”对象的其他函数或库。\(源码中是这样定义的：`file = tempfile.SpooledTemporaryFile(max_size=self.spool_max_size)`\)

- `UploadFile`的`file`属性具有很多异步方法\(这些方法也可以同步调用\)：

  - write\(data\)：写入文件

  - read\(size\)：读取文件，

    - 例：`contents = await myfile.read()`

  - seek\(offset\)：游标到文件中offset对应字节的位置

    - 例：`await myfile.seek(0)`将会到文件开始的位置
    - 如果运行一次`await myfile.read()`然后需要再次读取内容，则此功能特别有用。

  - close\(\)：关闭文件

#### 上传多个文件

```python
from typing import List

from fastapi import FastAPI, File, UploadFile
from fastapi.responses import HTMLResponse

app = FastAPI()


@app.post("/files/")
async def create_files(files: List[bytes] = File(...)):
    return {"file_sizes": [len(file) for file in files]}


@app.post("/uploadfiles/")
async def create_upload_files(files: List[UploadFile] = File(...)):
    return {"filenames": [file.filename for file in files]}


@app.get("/")
async def main():
    content = """
<body>
<form action="/files/" enctype="multipart/form-data" method="post">
<input name="files" type="file" multiple>
<input type="submit">
</form>
<form action="/uploadfiles/" enctype="multipart/form-data" method="post">
<input name="files" type="file" multiple>
<input type="submit">
</form>
</body>
    """
    return HTMLResponse(content=content)
```

可以打开`http://127.0.0.1:8000/docs`或者`http://127.0.0.1:8000/`进行尝试，请求`http://127.0.0.1:8000/`会进入到我们自定义的页面：  
![[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-uz9gyX1J-1599203933294)(evernotecid://FBE381A3-17C7-41D9-AA37-9C5F29FAB396/appyinxiangcom/20545635/ENResource/p220)]](https://img-blog.csdnimg.cn/20200904152000228.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDE1NjQ4Nw==,size_16,color_FFFFFF,t_70#pic_center)

## 总结

- 无论是使用`Form`还是`File`都不要忘记安装`python-multipart`
- 如果我们同时使用了`Form`和`Body`，请求将会以表单的形式发送给服务端
- 上传大文件时最好使用`UploadFile`
- 如果方法是异步的，可以使用`UploadFile`

> 上述栗子均放到git上啦，地址：[戳这里](https://github.com/ChuXiaoYi/fastapi)