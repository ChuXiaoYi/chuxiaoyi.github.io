---
title: django项目开发实战——博客(二)
date: 2018-12-18 17:22:11
tags: django django
categories: python
---


参考文档：  
\- [Django 博客使用 Markdown 自动生成文章目录](https://cloud.tencent.com/developer/article/1099709)  
\- [Django Haystack 全文检索与关键词高亮](https://www.zmrenwu.com/post/45/)

自从做了之前的[django项目开发实战——博客](https://blog.csdn.net/weixin_40156487/article/details/81512807)之后，一直没有继续做下去。现在终于有时间把后面的笔记补起来了！\(虽然还是一个很简陋滴= =\)

<!--more-->

git地址：<https://github.com/ChuXiaoYi/BlogWebSite/tree/test>

先放图：  
![image_1cv08semaqaa1d981p161p4n20km.png-164.9kB](https://img-blog.csdnimg.cn/2018121912245131)

![image_1cv091rb08sr1d3rvoj1vegm621j.png-234.6kB](https://img-blog.csdnimg.cn/2018121912245155)

## 列表页

页面是扒的[这个地址](https://templatemag.com/blog/)，然后做了一些修改

列表页主要说明一下**分页**和**flash**插件

#### 分页

判断是否有上一页下一页需要用`paginator`处理过的`post_list`

```python
view.py

limit = 3
paginator = Paginator(post, limit)
page = request.GET.get('page', 1)
result = paginator.page(page)
context = {
    "post_list": result,
    "page": page,
}
```

```html
list.html

<div class="pagination clearfix">
    <ul class="pager">
        {% if post_list.has_previous %}
            <li class="previous"><a href="?page={{ post_list.previous_page_number }}"><span
                    aria-hidden="true">&larr;</span> Newer</a></li>
        {% else %}
            <li class="previous disabled"><a href="#"><span aria-hidden="true">&larr;</span>
                Newer</a></li>
        {% endif %}
        {% if post_list.has_next %}
            <li class="next"><a href="?page={{ post_list.next_page_number }}">Older <span
                    aria-hidden="true">&rarr;</span></a></li>
        {% else %}
            <li class="next disabled"><a href="#">Older <span aria-hidden="true">&rarr;</span></a>
            </li>
        {% endif %}
    </ul>
</div>
```

#### flash插件

这是一个国际友人写的一个js脚本。放到页面中可以直接用。这里我把它处理成html代码直接放到页面中了。[原地址戳这里](http://chabudai.org/blog/?s=honehoneclock)

```html
<object classid="clsid:d27cdb6e-ae6d-11cf-96b8-444553540000"
        codebase="http://fpdownload.macromedia.com/pub/shockwave/cabs/flash/swflash.cab#version=8,0,0,0"
        width="100%" height="auto" id="honehoneclock" align="middle">
    <param name="allowScriptAccess" value="always">
    <param name="movie"
           value="http://chabudai.sakura.ne.jp/blogparts/honehoneclock/honehone_clock_wh.swf">
    <param name="quality" value="high">
    <param name="bgcolor" value="#ffffff">
    <param name="wmode" value="transparent">
    <param name="play" value="true">
    <embed wmode="transparent"
           src="http://chabudai.sakura.ne.jp/blogparts/honehoneclock/honehone_clock_wh.swf"
           quality="high" bgcolor="#ffffff" width="100%" height="auto" name="honehoneclock"
           align="middle" allowscriptaccess="always" type="application/x-shockwave-flash"
           pluginspage="http://www.macromedia.com/go/getflashplayer">
</object>
```

![image_1cv0acqik1534vs31h8s1gs91ts53q.png-12kB](https://img-blog.csdnimg.cn/2018121912245176)

如果想把背景改为透明的，修改`http://chabudai.sakura.ne.jp/blogparts/honehoneclock/honehone_clock_wh.swf`为`http://chabudai.sakura.ne.jp/blogparts/honehoneclock/honehone_clock_tr.swf`就可以啦～

## 详情页

这里主要有说明两个：`markdown生成目录`和`markdown高亮`

 1.     首先，安装第三方包

```python
pip3 install mardkown
```

 2.     在`view.py`中导入第三方包，并修改代码为：

```python
md = markdown.Markdown(extensions=[
            'markdown.extensions.extra',
            'markdown.extensions.codehilite',
            'markdown.extensions.toc',
        ])
post.body = md.convert(post.body)

comment_list = Comment.objects.filter(post__id=pk)

context = {
    'post': post,
    'toc': md.toc,
    'comment_list': comment_list
}
```

注意:这里使用的是`Markdown`实例化一个对象！

```python
'markdown.extensions.toc',
```

```python
'toc': md.toc,
```

如果想要自动生成目录，一定要写这两个！

但是如果我们点击目录，发现url变得很不美观，接下来我们来优化一下

将`markdown.extensions.toc`改为`TocExtension(slugify=slugify)`，但是在这之前，不要忘记引入两个必要的包`from django.utils.text import slugify`和`from markdown.extensions.toc import TocExtension`

```python
md = markdown.Markdown(extensions=[
        'markdown.extensions.extra',
        'markdown.extensions.codehilite',
        TocExtension(slugify=slugify),
    ])
```

`TocExtension`在实例化时其`slugify`参数可以接受一个函数作为参数，这个函数将被用于处理标题的锚点值。Markdown内置的处理方法不能处理中文标题，所以我们使用了`django.utils.text`中的`slugify`方法，该方法可以很好地处理中文.

现在在看就可以发现，url变得好看多了  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181219111501584.png)

但是，如果只是这样的话，并不能实现代码的高亮。接下来我们还需要做一些事情。

 3.     安装Pygments

```python
pip3 install Pygments 
```

运行`pygmentize \-S default \-f html \-a .codehilite > code.css`命令,将`code.css`放到你的静态文件目录下，并在html的头部添加`<link rel="stylesheet" href="{% static 'Post/css/code.css' %}">`\(你的路径可能和我的不一样，不要错啦！\)

最后，千万不要忘记添加

```python
'markdown.extensions.codehilite',
```

如果步骤正确，你会看到这样滴：  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181218183108452.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDE1NjQ4Nw==,size_16,color_FFFFFF,t_70)

我很喜欢CSDN可以隐藏过长的文章。我也实现了一个  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181218183358167.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDE1NjQ4Nw==,size_16,color_FFFFFF,t_70)

```html
<div class="hide-article-box text-center" id="hide-article-box">
   <a class="btn" id="btn-readmore" onclick="btnReadMore()">阅读更多</a>
</div>
```

```js
window.onload = function () {
        var readMore = document.getElementById('btn-readmore');
        var hideArticleBox = document.getElementById('hide-article-box');
        var content = document.getElementById('entry-content');
        readMore.onclick = function () {
            content.style.height = 'auto';
            hideArticleBox.style.display = 'none';
        }
    }
```

## 全文检索

对于一个搜索引擎来说，至少应该能够根据用户的搜索关键词对搜索结果进行排序以及高亮关键字。这里使用的是`django-haystack`实现这些特性。

django-haystack 是一个专门提供搜索功能的 django 第三方应用，它支持 Solr、Elasticsearch、Whoosh、Xapian 等多种搜索引擎，配合著名的中文自然语言处理库`jieba`分词，就可以为我们的博客提供一个效果不错的博客文章搜索系统。

**首先，安装依赖包：**

```python
pip3 install whoosh django-haystack jieba
```

注意：不要同时安装`haystack`和`django-haystack`，会冲突的！！

**配置setting**

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    # 这里要写哦～
    'haystack',
    "Post",
    "comment"
]
```

```python
HAYSTACK_CONNECTIONS = {
    'default': {
        # 指定了 django haystack 使用的搜索引擎，这里我们使用了 blog.whoosh_cn_backend.WhooshEngine，虽然目前这个引擎还不存在，但我们接下来会创建它。
        'ENGINE': 'Post.whoosh_cn_backend.WhooshEngine',
        # PATH 指定了索引文件需要存放的位置，我们设置为项目根目录 BASE_DIR 下的 whoosh_index 文件夹（在建立索引是会自动创建）
        'PATH': os.path.join(BASE_DIR, 'whoosh_index'),
    },
}
# 指定如何对搜索结果分页，这里设置为每 10 项结果为一页。
HAYSTACK_SEARCH_RESULTS_PER_PAGE = 10
# 指定什么时候更新索引，这里我们使用 haystack.signals.RealtimeSignalProcessor，作用是每当有文章更新时就更新索引。由于博客文章更新不会太频繁，因此实时更新没有问题。
HAYSTACK_SIGNAL_PROCESSOR = 'haystack.signals.RealtimeSignalProcessor'
```

**处理数据**

接下来就要告诉 django haystack 使用那些数据建立索引以及如何存放索引。如果要对`Post`应用下的数据进行全文检索，做法是在`Post`应用下建立一个`search_indexes.py`文件，写上如下代码：

```python
from haystack import indexes
from .models import Post


class PostIndex(indexes.SearchIndex, indexes.Indexable):
    text = indexes.CharField(document=True, use_template=True)

    def get_model(self):
        return Post

    def index_queryset(self, using=None):
        return self.get_model().objects.all()
```

路径：  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181219120855412.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDE1NjQ4Nw==,size_16,color_FFFFFF,t_70)

这是 django haystack 的规定。要相对某个 app 下的数据进行全文检索，就要在该 app 下创建一个 search\_indexes.py 文件，然后创建一个 XXIndex 类（XX 为含有被检索数据的模型，如这里的 Post），并且继承 SearchIndex 和 Indexable。

每个索引里面必须有且只能有一个字段为`document=True`，这代表 django haystack 和搜索引擎将使用此字段的内容作为索引进行检索\(primary field\)。注意，如果使用一个字段设置了`document=True`，则一般约定此字段名为`text`，这是在 SearchIndex 类里面一贯的命名，以防止后台混乱，当然名字你也可以随便改，不过不建议改。

并且，haystack提供了`use_template=True`在text字段中这样就允许我们使用数据模板去建立搜索引擎索引的文件，说得通俗点就是索引里面需要存放一些什么东西，例如Post的title字段，这样我们可以通过 title 内容来检索 Post 数据了。举个例子，假如你搜索 Python ，那么就可以检索出title中含有 Python 的Post了，怎么样是不是很简单？数据模板的路径为 `templates/search/indexes/youapp/\<model_name>_text.txt`（例如 templates/search/indexes/blog/post\_text.txt, model\_name必须是小写的哦！\)，其内容为：

```python
{{ object.title }}
{{ object.body }}
```

路径：  
![在这里插入图片描述](https://img-blog.csdnimg.cn/2018121912092277.png)

**配置 URL**  
在项目的`urls.py`中添加

```python
urlpatterns = [
    re_path('admin/', admin.site.urls),
    # 在这里
    re_path(r'^search/', include('haystack.urls')),
    re_path(r'', include('Post.urls', namespace="Post")),
    re_path(r'', include('comment.urls', namespace="comment")),
]
```

**修改搜索表单**

```html
<form class="input-group" action="{% url 'haystack_search' %}">
    <input class="form-control" type="search" name="q" placeholder="search">
    <span class="input-group-btn">
        <button class="btn btn-default" type="submit">Go!</button>
    </span>
</form>
```

注意：input的name属性的值一定要是`q`

**创建搜索结果页面**  
`haystack_search`视图函数会将搜索结果传递给模板`search/search.html`，因此创建这个模板文件，对搜索结果进行渲染：

```html
{% if query %}
    {% for result in page.object_list %}
        <article id="post-77"
                 class="clearfix post post-77 blog type-blog status-publish has-post-thumbnail hentry">
            <h2 class="entry-title">
                <a href="{% url 'Post:detail' result.object.id %}" rel="bookmark">
                    {% highlight result.object.title with query %}
                </a>
            </h2>
            <div class="entry-meta"><i class="glyphicon glyphicon-time"></i>
                <time class="meta-date" itemprop="datePublished"
                      datetime="{{ result.object.modified_time }}"> {{ result.object.modified_time }}</time>
                <i class="glyphicon glyphicon-tags"></i>&nbsp;
                <span class="post-cate">{{ result.object.category.name }}</span>
            </div>
            <div class="entry-content">
                <p>{% highlight result.object.body with query %}&hellip;</p>
            </div>
            <div class="etry-more">
                <a href="{% url 'Post:detail' result.object.id %}" rel="bookmark">Read More</a>
            </div>
        </article>
    {% empty %}
        <div class="no-post">没有搜索到你想要的结果！</div>
    {% endfor %}
{% endif %}
```

路径：  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181219121428191.png)

这个模板基本和`Post/list.html`一样，只是由于haystack对搜索结果做了分页，传给模板的变量是一个`page`对象，所以我们从`page`中取出这一页对应的搜索结果，然后对其循环显示，即`{% for result in page.object_list %}`。另外要取得Post（文章）以显示文章的数据如标题、正文，需要从`result`的`object`属性中获取.`query`变量的值即为用户搜索的关键词。

**高亮关键词**  
在django haystack中实现这个效果也非常简单，只需要使用`{% highlight %}`模板标签即可,上面的示例已经有了高亮。高亮处理的原理其实就是给文本中的关键字包上一个`span`标签并且为其添加`highlighted`样式,因此你可以自己设置高亮的颜色哦：

```css
<style>
    span.highlighted {
        color: red;
    }
</style>
```

**修改搜索引擎为中文分词**  
我们使用 Whoosh 作为搜索引擎，但在 django haystack 中为 Whoosh 指定的分词器是英文分词器，可能会使得搜索结果不理想，我们把这个分词器替换成 jieba 中文分词器。从你安装的 haystack 中把`haystack/backends/whoosh_backends.py`文件拷贝到`Post/`下，重命名为`whoosh_cn_backends.py`（之前我们在 `settings.py`中的`HAYSTACK_CONNECTIONS` 指定的就是这个文件），然后找到如下一行代码：

```python
schema_fields[field_class.index_fieldname] = TEXT(stored=True, analyzer=StemmingAnalyzer(), field_boost=field_class.boost, sortable=True)
```

将其中的`analyzer`改为`ChineseAnalyzer`，当然为了使用它，你需要在文件顶部引入：`from jieba.analyse import ChineseAnalyzer`。  
改成这样：

```python
#注意先找到这个再修改，而不是直接添加  
schema_fields[field_class.index_fieldname] = TEXT(stored=True, analyzer=ChineseAnalyzer(),field_boost=field_class.boost, sortable=True)
```

**建立索引文件**  
最后一步就是建立索引文件了，运行命令`python manage.py rebuild_index`就可以建立索引文件了。  
最后就可以看到效果啦  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181219122427497.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDE1NjQ4Nw==,size_16,color_FFFFFF,t_70)
