---
title: fastapi教程-进阶九（Dependencies-1）
date: 2020-09-18 15:56:23
tags: python fastapi
categories: fastapi
---



**参考内容**：

- <https://fastapi.tiangolo.com/>

fastapi有非常强大的**依赖注入**系统，虽然听起来会很难，但是用起来非常简单，并且非常方便我们集成各种组件

<!--more-->

## 什么是依赖注入

依赖注入是指在我们的项目中，定义了一些方法，这些方法是我们某些路径方法需要依赖的，这些方法叫做`依赖项`，当代码运行时，fastapi会将这个依赖项**注入**到路径方法中。

如果很难理解，我们可以通过了解他的使用场景来理解他的意思。依赖注入可以用在下面的场景：

- 有共享逻辑时，也就是重复的业务逻辑
- 共享数据连接
- 安全机制、权限校验、角色管理等等
- 其他的可以复用或重复的逻辑

通过使用依赖注入，可以大大减少重复代码。

## 快速入门

我们先来看一个简单的例子，从而理解fastapi的依赖注入：

```python
from typing import Optional

from fastapi import Depends, FastAPI

app = FastAPI()


async def common_parameters(q: Optional[str] = None, skip: int = 0, limit: int = 100):
    return {"q": q, "skip": skip, "limit": limit}


@app.get("/items/")
async def read_items(commons: dict = Depends(common_parameters)):
    return commons


@app.get("/users/")
async def read_users(commons: dict = Depends(common_parameters)):
    return commons
```

在这里个例子中，我们可以发现`read_items`和`read_users`这两个路径方法都使用了`common_parameters`这个依赖项，这个依赖项希望得到：

- string类型的q参数
- int类型的，默认值是0的skip参数
- int类型，默认值是0的limit参数

并且，这个依赖项会返回一个由参数组成的dict

> 需要注意的一点，`common_parameters`是一个方法，是一个可调用对象，也就意味着`Depends`需要接收一个可调用对象作为参数

我们先来启动服务，打开`http://127.0.0.1:8000/docs`，看一看她会生成怎样的api：

![[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-mwaIPrBF-1600415650020)(evernotecid://FBE381A3-17C7-41D9-AA37-9C5F29FAB396/appyinxiangcom/20545635/ENResource/p227)]](https://img-blog.csdnimg.cn/2020091815543814.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDE1NjQ4Nw==,size_16,color_FFFFFF,t_70#pic_center)  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200918155449869.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDE1NjQ4Nw==,size_16,color_FFFFFF,t_70#pic_center)

不难发现，api接收的参数就是依赖项的参数，到这里，我们尝试一下请求这两个api，其实我们可以分析出来当一个请求进来时fastapi是如何使用依赖注入的了：

1.  先调用依赖项这个方法，并传入正确的参数
2.  执行依赖项，得到返回
3.  将依赖项返回结果指派给路径方法对应的参数

如果很难理解，可以看下面这个图：

<style>#mermaid-svg-s00DO82jbQvVmlVK .label{font-family:'trebuchet ms', verdana, arial;font-family:var(--mermaid-font-family);fill:#333;color:#333}#mermaid-svg-s00DO82jbQvVmlVK .label text{fill:#333}#mermaid-svg-s00DO82jbQvVmlVK .node rect,#mermaid-svg-s00DO82jbQvVmlVK .node circle,#mermaid-svg-s00DO82jbQvVmlVK .node ellipse,#mermaid-svg-s00DO82jbQvVmlVK .node polygon,#mermaid-svg-s00DO82jbQvVmlVK .node path{fill:#ECECFF;stroke:#9370db;stroke-width:1px}#mermaid-svg-s00DO82jbQvVmlVK .node .label{text-align:center;fill:#333}#mermaid-svg-s00DO82jbQvVmlVK .node.clickable{cursor:pointer}#mermaid-svg-s00DO82jbQvVmlVK .arrowheadPath{fill:#333}#mermaid-svg-s00DO82jbQvVmlVK .edgePath .path{stroke:#333;stroke-width:1.5px}#mermaid-svg-s00DO82jbQvVmlVK .flowchart-link{stroke:#333;fill:none}#mermaid-svg-s00DO82jbQvVmlVK .edgeLabel{background-color:#e8e8e8;text-align:center}#mermaid-svg-s00DO82jbQvVmlVK .edgeLabel rect{opacity:0.9}#mermaid-svg-s00DO82jbQvVmlVK .edgeLabel span{color:#333}#mermaid-svg-s00DO82jbQvVmlVK .cluster rect{fill:#ffffde;stroke:#aa3;stroke-width:1px}#mermaid-svg-s00DO82jbQvVmlVK .cluster text{fill:#333}#mermaid-svg-s00DO82jbQvVmlVK div.mermaidTooltip{position:absolute;text-align:center;max-width:200px;padding:2px;font-family:'trebuchet ms', verdana, arial;font-family:var(--mermaid-font-family);font-size:12px;background:#ffffde;border:1px solid #aa3;border-radius:2px;pointer-events:none;z-index:100}#mermaid-svg-s00DO82jbQvVmlVK .actor{stroke:#ccf;fill:#ECECFF}#mermaid-svg-s00DO82jbQvVmlVK text.actor>tspan{fill:#000;stroke:none}#mermaid-svg-s00DO82jbQvVmlVK .actor-line{stroke:grey}#mermaid-svg-s00DO82jbQvVmlVK .messageLine0{stroke-width:1.5;stroke-dasharray:none;stroke:#333}#mermaid-svg-s00DO82jbQvVmlVK .messageLine1{stroke-width:1.5;stroke-dasharray:2, 2;stroke:#333}#mermaid-svg-s00DO82jbQvVmlVK #arrowhead path{fill:#333;stroke:#333}#mermaid-svg-s00DO82jbQvVmlVK .sequenceNumber{fill:#fff}#mermaid-svg-s00DO82jbQvVmlVK #sequencenumber{fill:#333}#mermaid-svg-s00DO82jbQvVmlVK #crosshead path{fill:#333;stroke:#333}#mermaid-svg-s00DO82jbQvVmlVK .messageText{fill:#333;stroke:#333}#mermaid-svg-s00DO82jbQvVmlVK .labelBox{stroke:#ccf;fill:#ECECFF}#mermaid-svg-s00DO82jbQvVmlVK .labelText,#mermaid-svg-s00DO82jbQvVmlVK .labelText>tspan{fill:#000;stroke:none}#mermaid-svg-s00DO82jbQvVmlVK .loopText,#mermaid-svg-s00DO82jbQvVmlVK .loopText>tspan{fill:#000;stroke:none}#mermaid-svg-s00DO82jbQvVmlVK .loopLine{stroke-width:2px;stroke-dasharray:2, 2;stroke:#ccf;fill:#ccf}#mermaid-svg-s00DO82jbQvVmlVK .note{stroke:#aa3;fill:#fff5ad}#mermaid-svg-s00DO82jbQvVmlVK .noteText,#mermaid-svg-s00DO82jbQvVmlVK .noteText>tspan{fill:#000;stroke:none}#mermaid-svg-s00DO82jbQvVmlVK .activation0{fill:#f4f4f4;stroke:#666}#mermaid-svg-s00DO82jbQvVmlVK .activation1{fill:#f4f4f4;stroke:#666}#mermaid-svg-s00DO82jbQvVmlVK .activation2{fill:#f4f4f4;stroke:#666}#mermaid-svg-s00DO82jbQvVmlVK .mermaid-main-font{font-family:"trebuchet ms", verdana, arial;font-family:var(--mermaid-font-family)}#mermaid-svg-s00DO82jbQvVmlVK .section{stroke:none;opacity:0.2}#mermaid-svg-s00DO82jbQvVmlVK .section0{fill:rgba(102,102,255,0.49)}#mermaid-svg-s00DO82jbQvVmlVK .section2{fill:#fff400}#mermaid-svg-s00DO82jbQvVmlVK .section1,#mermaid-svg-s00DO82jbQvVmlVK .section3{fill:#fff;opacity:0.2}#mermaid-svg-s00DO82jbQvVmlVK .sectionTitle0{fill:#333}#mermaid-svg-s00DO82jbQvVmlVK .sectionTitle1{fill:#333}#mermaid-svg-s00DO82jbQvVmlVK .sectionTitle2{fill:#333}#mermaid-svg-s00DO82jbQvVmlVK .sectionTitle3{fill:#333}#mermaid-svg-s00DO82jbQvVmlVK .sectionTitle{text-anchor:start;font-size:11px;text-height:14px;font-family:'trebuchet ms', verdana, arial;font-family:var(--mermaid-font-family)}#mermaid-svg-s00DO82jbQvVmlVK .grid .tick{stroke:#d3d3d3;opacity:0.8;shape-rendering:crispEdges}#mermaid-svg-s00DO82jbQvVmlVK .grid .tick text{font-family:'trebuchet ms', verdana, arial;font-family:var(--mermaid-font-family)}#mermaid-svg-s00DO82jbQvVmlVK .grid path{stroke-width:0}#mermaid-svg-s00DO82jbQvVmlVK .today{fill:none;stroke:red;stroke-width:2px}#mermaid-svg-s00DO82jbQvVmlVK .task{stroke-width:2}#mermaid-svg-s00DO82jbQvVmlVK .taskText{text-anchor:middle;font-family:'trebuchet ms', verdana, arial;font-family:var(--mermaid-font-family)}#mermaid-svg-s00DO82jbQvVmlVK .taskText:not([font-size]){font-size:11px}#mermaid-svg-s00DO82jbQvVmlVK .taskTextOutsideRight{fill:#000;text-anchor:start;font-size:11px;font-family:'trebuchet ms', verdana, arial;font-family:var(--mermaid-font-family)}#mermaid-svg-s00DO82jbQvVmlVK .taskTextOutsideLeft{fill:#000;text-anchor:end;font-size:11px}#mermaid-svg-s00DO82jbQvVmlVK .task.clickable{cursor:pointer}#mermaid-svg-s00DO82jbQvVmlVK .taskText.clickable{cursor:pointer;fill:#003163 !important;font-weight:bold}#mermaid-svg-s00DO82jbQvVmlVK .taskTextOutsideLeft.clickable{cursor:pointer;fill:#003163 !important;font-weight:bold}#mermaid-svg-s00DO82jbQvVmlVK .taskTextOutsideRight.clickable{cursor:pointer;fill:#003163 !important;font-weight:bold}#mermaid-svg-s00DO82jbQvVmlVK .taskText0,#mermaid-svg-s00DO82jbQvVmlVK .taskText1,#mermaid-svg-s00DO82jbQvVmlVK .taskText2,#mermaid-svg-s00DO82jbQvVmlVK .taskText3{fill:#fff}#mermaid-svg-s00DO82jbQvVmlVK .task0,#mermaid-svg-s00DO82jbQvVmlVK .task1,#mermaid-svg-s00DO82jbQvVmlVK .task2,#mermaid-svg-s00DO82jbQvVmlVK .task3{fill:#8a90dd;stroke:#534fbc}#mermaid-svg-s00DO82jbQvVmlVK .taskTextOutside0,#mermaid-svg-s00DO82jbQvVmlVK .taskTextOutside2{fill:#000}#mermaid-svg-s00DO82jbQvVmlVK .taskTextOutside1,#mermaid-svg-s00DO82jbQvVmlVK .taskTextOutside3{fill:#000}#mermaid-svg-s00DO82jbQvVmlVK .active0,#mermaid-svg-s00DO82jbQvVmlVK .active1,#mermaid-svg-s00DO82jbQvVmlVK .active2,#mermaid-svg-s00DO82jbQvVmlVK .active3{fill:#bfc7ff;stroke:#534fbc}#mermaid-svg-s00DO82jbQvVmlVK .activeText0,#mermaid-svg-s00DO82jbQvVmlVK .activeText1,#mermaid-svg-s00DO82jbQvVmlVK .activeText2,#mermaid-svg-s00DO82jbQvVmlVK .activeText3{fill:#000 !important}#mermaid-svg-s00DO82jbQvVmlVK .done0,#mermaid-svg-s00DO82jbQvVmlVK .done1,#mermaid-svg-s00DO82jbQvVmlVK .done2,#mermaid-svg-s00DO82jbQvVmlVK .done3{stroke:grey;fill:#d3d3d3;stroke-width:2}#mermaid-svg-s00DO82jbQvVmlVK .doneText0,#mermaid-svg-s00DO82jbQvVmlVK .doneText1,#mermaid-svg-s00DO82jbQvVmlVK .doneText2,#mermaid-svg-s00DO82jbQvVmlVK .doneText3{fill:#000 !important}#mermaid-svg-s00DO82jbQvVmlVK .crit0,#mermaid-svg-s00DO82jbQvVmlVK .crit1,#mermaid-svg-s00DO82jbQvVmlVK .crit2,#mermaid-svg-s00DO82jbQvVmlVK .crit3{stroke:#f88;fill:red;stroke-width:2}#mermaid-svg-s00DO82jbQvVmlVK .activeCrit0,#mermaid-svg-s00DO82jbQvVmlVK .activeCrit1,#mermaid-svg-s00DO82jbQvVmlVK .activeCrit2,#mermaid-svg-s00DO82jbQvVmlVK .activeCrit3{stroke:#f88;fill:#bfc7ff;stroke-width:2}#mermaid-svg-s00DO82jbQvVmlVK .doneCrit0,#mermaid-svg-s00DO82jbQvVmlVK .doneCrit1,#mermaid-svg-s00DO82jbQvVmlVK .doneCrit2,#mermaid-svg-s00DO82jbQvVmlVK .doneCrit3{stroke:#f88;fill:#d3d3d3;stroke-width:2;cursor:pointer;shape-rendering:crispEdges}#mermaid-svg-s00DO82jbQvVmlVK .milestone{transform:rotate(45deg) scale(0.8, 0.8)}#mermaid-svg-s00DO82jbQvVmlVK .milestoneText{font-style:italic}#mermaid-svg-s00DO82jbQvVmlVK .doneCritText0,#mermaid-svg-s00DO82jbQvVmlVK .doneCritText1,#mermaid-svg-s00DO82jbQvVmlVK .doneCritText2,#mermaid-svg-s00DO82jbQvVmlVK .doneCritText3{fill:#000 !important}#mermaid-svg-s00DO82jbQvVmlVK .activeCritText0,#mermaid-svg-s00DO82jbQvVmlVK .activeCritText1,#mermaid-svg-s00DO82jbQvVmlVK .activeCritText2,#mermaid-svg-s00DO82jbQvVmlVK .activeCritText3{fill:#000 !important}#mermaid-svg-s00DO82jbQvVmlVK .titleText{text-anchor:middle;font-size:18px;fill:#000;font-family:'trebuchet ms', verdana, arial;font-family:var(--mermaid-font-family)}#mermaid-svg-s00DO82jbQvVmlVK g.classGroup text{fill:#9370db;stroke:none;font-family:'trebuchet ms', verdana, arial;font-family:var(--mermaid-font-family);font-size:10px}#mermaid-svg-s00DO82jbQvVmlVK g.classGroup text .title{font-weight:bolder}#mermaid-svg-s00DO82jbQvVmlVK g.clickable{cursor:pointer}#mermaid-svg-s00DO82jbQvVmlVK g.classGroup rect{fill:#ECECFF;stroke:#9370db}#mermaid-svg-s00DO82jbQvVmlVK g.classGroup line{stroke:#9370db;stroke-width:1}#mermaid-svg-s00DO82jbQvVmlVK .classLabel .box{stroke:none;stroke-width:0;fill:#ECECFF;opacity:0.5}#mermaid-svg-s00DO82jbQvVmlVK .classLabel .label{fill:#9370db;font-size:10px}#mermaid-svg-s00DO82jbQvVmlVK .relation{stroke:#9370db;stroke-width:1;fill:none}#mermaid-svg-s00DO82jbQvVmlVK .dashed-line{stroke-dasharray:3}#mermaid-svg-s00DO82jbQvVmlVK #compositionStart{fill:#9370db;stroke:#9370db;stroke-width:1}#mermaid-svg-s00DO82jbQvVmlVK #compositionEnd{fill:#9370db;stroke:#9370db;stroke-width:1}#mermaid-svg-s00DO82jbQvVmlVK #aggregationStart{fill:#ECECFF;stroke:#9370db;stroke-width:1}#mermaid-svg-s00DO82jbQvVmlVK #aggregationEnd{fill:#ECECFF;stroke:#9370db;stroke-width:1}#mermaid-svg-s00DO82jbQvVmlVK #dependencyStart{fill:#9370db;stroke:#9370db;stroke-width:1}#mermaid-svg-s00DO82jbQvVmlVK #dependencyEnd{fill:#9370db;stroke:#9370db;stroke-width:1}#mermaid-svg-s00DO82jbQvVmlVK #extensionStart{fill:#9370db;stroke:#9370db;stroke-width:1}#mermaid-svg-s00DO82jbQvVmlVK #extensionEnd{fill:#9370db;stroke:#9370db;stroke-width:1}#mermaid-svg-s00DO82jbQvVmlVK .commit-id,#mermaid-svg-s00DO82jbQvVmlVK .commit-msg,#mermaid-svg-s00DO82jbQvVmlVK .branch-label{fill:lightgrey;color:lightgrey;font-family:'trebuchet ms', verdana, arial;font-family:var(--mermaid-font-family)}#mermaid-svg-s00DO82jbQvVmlVK .pieTitleText{text-anchor:middle;font-size:25px;fill:#000;font-family:'trebuchet ms', verdana, arial;font-family:var(--mermaid-font-family)}#mermaid-svg-s00DO82jbQvVmlVK .slice{font-family:'trebuchet ms', verdana, arial;font-family:var(--mermaid-font-family)}#mermaid-svg-s00DO82jbQvVmlVK g.stateGroup text{fill:#9370db;stroke:none;font-size:10px;font-family:'trebuchet ms', verdana, arial;font-family:var(--mermaid-font-family)}#mermaid-svg-s00DO82jbQvVmlVK g.stateGroup text{fill:#9370db;fill:#333;stroke:none;font-size:10px}#mermaid-svg-s00DO82jbQvVmlVK g.statediagram-cluster .cluster-label text{fill:#333}#mermaid-svg-s00DO82jbQvVmlVK g.stateGroup .state-title{font-weight:bolder;fill:#000}#mermaid-svg-s00DO82jbQvVmlVK g.stateGroup rect{fill:#ECECFF;stroke:#9370db}#mermaid-svg-s00DO82jbQvVmlVK g.stateGroup line{stroke:#9370db;stroke-width:1}#mermaid-svg-s00DO82jbQvVmlVK .transition{stroke:#9370db;stroke-width:1;fill:none}#mermaid-svg-s00DO82jbQvVmlVK .stateGroup .composit{fill:white;border-bottom:1px}#mermaid-svg-s00DO82jbQvVmlVK .stateGroup .alt-composit{fill:#e0e0e0;border-bottom:1px}#mermaid-svg-s00DO82jbQvVmlVK .state-note{stroke:#aa3;fill:#fff5ad}#mermaid-svg-s00DO82jbQvVmlVK .state-note text{fill:black;stroke:none;font-size:10px}#mermaid-svg-s00DO82jbQvVmlVK .stateLabel .box{stroke:none;stroke-width:0;fill:#ECECFF;opacity:0.7}#mermaid-svg-s00DO82jbQvVmlVK .edgeLabel text{fill:#333}#mermaid-svg-s00DO82jbQvVmlVK .stateLabel text{fill:#000;font-size:10px;font-weight:bold;font-family:'trebuchet ms', verdana, arial;font-family:var(--mermaid-font-family)}#mermaid-svg-s00DO82jbQvVmlVK .node circle.state-start{fill:black;stroke:black}#mermaid-svg-s00DO82jbQvVmlVK .node circle.state-end{fill:black;stroke:white;stroke-width:1.5}#mermaid-svg-s00DO82jbQvVmlVK #statediagram-barbEnd{fill:#9370db}#mermaid-svg-s00DO82jbQvVmlVK .statediagram-cluster rect{fill:#ECECFF;stroke:#9370db;stroke-width:1px}#mermaid-svg-s00DO82jbQvVmlVK .statediagram-cluster rect.outer{rx:5px;ry:5px}#mermaid-svg-s00DO82jbQvVmlVK .statediagram-state .divider{stroke:#9370db}#mermaid-svg-s00DO82jbQvVmlVK .statediagram-state .title-state{rx:5px;ry:5px}#mermaid-svg-s00DO82jbQvVmlVK .statediagram-cluster.statediagram-cluster .inner{fill:white}#mermaid-svg-s00DO82jbQvVmlVK .statediagram-cluster.statediagram-cluster-alt .inner{fill:#e0e0e0}#mermaid-svg-s00DO82jbQvVmlVK .statediagram-cluster .inner{rx:0;ry:0}#mermaid-svg-s00DO82jbQvVmlVK .statediagram-state rect.basic{rx:5px;ry:5px}#mermaid-svg-s00DO82jbQvVmlVK .statediagram-state rect.divider{stroke-dasharray:10,10;fill:#efefef}#mermaid-svg-s00DO82jbQvVmlVK .note-edge{stroke-dasharray:5}#mermaid-svg-s00DO82jbQvVmlVK .statediagram-note rect{fill:#fff5ad;stroke:#aa3;stroke-width:1px;rx:0;ry:0}:root{--mermaid-font-family: '"trebuchet ms", verdana, arial';--mermaid-font-family: "Comic Sans MS", "Comic Sans", cursive}#mermaid-svg-s00DO82jbQvVmlVK .error-icon{fill:#522}#mermaid-svg-s00DO82jbQvVmlVK .error-text{fill:#522;stroke:#522}#mermaid-svg-s00DO82jbQvVmlVK .edge-thickness-normal{stroke-width:2px}#mermaid-svg-s00DO82jbQvVmlVK .edge-thickness-thick{stroke-width:3.5px}#mermaid-svg-s00DO82jbQvVmlVK .edge-pattern-solid{stroke-dasharray:0}#mermaid-svg-s00DO82jbQvVmlVK .edge-pattern-dashed{stroke-dasharray:3}#mermaid-svg-s00DO82jbQvVmlVK .edge-pattern-dotted{stroke-dasharray:2}#mermaid-svg-s00DO82jbQvVmlVK .marker{fill:#333}#mermaid-svg-s00DO82jbQvVmlVK .marker.cross{stroke:#333} :root { --mermaid-font-family: "trebuchet ms", verdana, arial;}</style> <style>#mermaid-svg-s00DO82jbQvVmlVK { color: rgba(0, 0, 0, 0.75); font: ; }</style>

common\_parameters

/items/

/users/

> 通过这个入门可以快速学习到如何使用依赖注入，我们可以得出一个结论：在使用依赖注入时，我们不需要做其他的事情，只需要将依赖项定义好，并告诉fastapi哪些路径方法需要使用这个依赖项，其他的都交给fastapi就好，当请求进来后，fastapi会负责将依赖项注入到方法中。

## Fastapi兼容性

依赖注入的简单性使得fastapi具备了更好的兼容性，通过依赖注入fastapi可以兼容：

- 所有关系数据库
- NoSQL数据库
- 外部包装
- 外部API
- 认证和授权系统
- API使用情况监控系统
- 响应数据注入系统
- 等等

## 用类做依赖项

在上面的快速入门中，我们学习了如何用方法来做依赖项，我们也提到了，`Depends`需要接收一个可调用对象作为参数，那么也就是说，**依赖项必须是可调用对象**。下面我们尝试用`类`来做依赖项：

```python
from typing import Optional

from fastapi import Depends, FastAPI

app = FastAPI()


fake_items_db = [{"item_name": "Foo"}, {"item_name": "Bar"}, {"item_name": "Baz"}]


class CommonQueryParams:
    def __init__(self, q: Optional[str] = None, skip: int = 0, limit: int = 100):
        self.q = q
        self.skip = skip
        self.limit = limit


@app.get("/items/")
async def read_items(commons: CommonQueryParams = Depends(CommonQueryParams)):
    response = {}
    if commons.q:
        response.update({"q": commons.q})
    items = fake_items_db[commons.skip : commons.skip + commons.limit]
    response.update({"items": items})
    return response
```

注意看`CommonQueryParams`类的`__init__`方法，他接收的参数和上面的`common_parameters`方法是一样的：

```python
def __init__(self, q: Optional[str] = None, skip: int = 0, limit: int = 100):
```

然后我们告诉fastapi接口的依赖项是`CommonQueryParams`类:

```python
async def read_items(commons: CommonQueryParams = Depends(CommonQueryParams)):
```

当请求进来后，fastapi会调用`CommonQueryParams`类，并创建一个实例，将这个实例赋值给`commons`参数。

#### depends简化写法

在上面的例子中，我们注意到`CommonQueryParams`写了两遍:

```python
commons: CommonQueryParams = Depends(CommonQueryParams)
```

其实，第一个CommonQueryParams并没有什么特殊含义，fastapi也不会用它来做数据验证，而第二个CommonQueryParams才是fastapi会用到的依赖项，因此，我们可以简化成：

```python
commons=Depends(CommonQueryParams)
```

此时的代码就变成了：

```python
from typing import Optional

from fastapi import Depends, FastAPI

app = FastAPI()

fake_items_db = [{"item_name": "Foo"}, {"item_name": "Bar"}, {"item_name": "Baz"}]


class CommonQueryParams:
    def __init__(self, q: Optional[str] = None, skip: int = 0, limit: int = 100):
        self.q = q
        self.skip = skip
        self.limit = limit


@app.get("/items/")
async def read_items(commons=Depends(CommonQueryParams)):
    response = {}
    if commons.q:
        response.update({"q": commons.q})
    items = fake_items_db[commons.skip: commons.skip + commons.limit]
    response.update({"items": items})
    return response
```

但是，fastapi也鼓励我们用`commons: CommonQueryParams = Depends(CommonQueryParams)`这样的形式来声明参数类型，这样写我们的编译器可以知道这个参数的类型，从而帮助我们进行代码补全或检查:  
![[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-Lhbcx4iH-1600415650024)(evernotecid://FBE381A3-17C7-41D9-AA37-9C5F29FAB396/appyinxiangcom/20545635/ENResource/p229)]](https://img-blog.csdnimg.cn/20200918155513716.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDE1NjQ4Nw==,size_16,color_FFFFFF,t_70#pic_center)

那么如何同时具备这两个特点呢，我们可以这样写：

```python
commons: CommonQueryParams = Depends()
```

此时的代码变成了：

```python
from typing import Optional

from fastapi import Depends, FastAPI

app = FastAPI()

fake_items_db = [{"item_name": "Foo"}, {"item_name": "Bar"}, {"item_name": "Baz"}]


class CommonQueryParams:
    def __init__(self, q: Optional[str] = None, skip: int = 0, limit: int = 100):
        self.q = q
        self.skip = skip
        self.limit = limit


@app.get("/items/")
async def read_items(commons: CommonQueryParams = Depends()):
    response = {}

    if commons.q:
        response.update({"q": commons.q})
    items = fake_items_db[commons.skip: commons.skip + commons.limit]
    response.update({"items": items})
    return response
```

> 上述栗子均放到git上啦，地址：[戳这里](https://github.com/ChuXiaoYi/fastapi)
