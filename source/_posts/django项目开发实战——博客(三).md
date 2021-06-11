---
title: django项目开发实战——博客(三)
date: 2019-01-09 18:32:22
tags: django celery django
categories: python
---

经过好几版的修改，终于变成了现在这个样子（咳咳。）网站可以[戳这里](http://chuxiaoyi.cn/)，欢迎大家多提意见哦～

<!--more-->

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190109171946399.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDE1NjQ4Nw==,size_16,color_FFFFFF,t_70)

在之前的基础上，做了以下增加：

## 评论功能

评论功能依旧用了渲染的方式，加了一些原生js控制css样式。

评论框：

```html
<form class="pure-form comment" method="post" action="{% url 'comment:comment' pk=post.id %}">
    <fieldset class="pure-group">
        昵称：<input class="pure-input-1-4" name="comment-name" type="text" placeholder="昵称">
        &nbsp;&nbsp;
        邮箱：<input class="pure-input-1-3" name="comment-email" type="email" placeholder="留下邮箱，可以收到回复提醒哦～">
    </fieldset>
    <textarea class="comment-content" name="comment" placeholder="想对作者说点什么。。。" onfocus="show_person_info(this)"></textarea>
    <div class="comment-div">
        <span id="tip_comment">支持markdown格式</span>
        <button class="pure-button" type="submit" href="#">发表评论</button>
    </div>
</form>
```

评论展示区域

```html
{% for comment in comment_list %}
   <ul class="comment-list ">
       <li class="comment-li first-comment" onmouseover="show_reply(this)"
           onmouseleave="hidden_replay(this)">
           <i class="fa fa-user-circle"></i>
           <span>{{ comment.name }}：</span>
           <a class="reply" href='javascript:void(0);' onclick="show_replyform(this)">回复</a>
           <div class="comment-info">{{ comment.text | safe }}</div>

           <form class="pure-form reply-form" id="reply-form" method="post"
                 action="{% url 'comment:comment' pk=post.id %}">

               <input hidden name="reply_to" value="{{ comment.id }}">
               <input hidden name="reply_name" value="{{ comment.name }}">
               <input hidden name="root_to" value="{{ comment.id }}">
               <input hidden name="reply_email" value="{{ comment.email }}">
               <fieldset class="pure-group">
                   昵称：<input class="pure-input-1-4" name="comment-name" type="text" placeholder="昵称">
                   &nbsp;&nbsp;
                   邮箱：<input class="pure-input-1-3" name="comment-email" type="email"
                             placeholder="留下邮箱，可以收到回复提醒哦～">
               </fieldset>
               <textarea class="comment-content" name="comment" onblur="hidden_replyform(this)"
                         placeholder="想对作者说点什么。。。"></textarea>
               <div class="comment-div">
                   <span id="tip_comment">支持markdown格式</span>
                   <button class="pure-button" type="submit" href="#">发表评论</button>
               </div>
           </form>

       </li>
       {% for reply in comment.replies %}
           <li class="comment-li reply-comment" onmouseover="show_reply(this)"
               onmouseleave="hidden_replay(this)">
               <i class="fa fa-user-circle"></i>
               <span>{{ reply.name }}回复{{ reply.reply_name }}: </span>
               <a class="reply" href='javascript:void(0);' onclick="show_replyform(this)">回复</a>
               <div class="comment-info">{{ reply.text | safe }}</div>

               <form class="pure-form reply-form" method="post"
                     action="{% url 'comment:comment' pk=post.id %}">
                   <input hidden name="reply_to" value="{{ reply.id }}">
                   <input hidden name="reply_name" value="{{ reply.name }}">
                   <input hidden name="root_to" value="{{ comment.id }}">
                   <input hidden name="reply_email" value="{{ reply.email }}">
                   <fieldset class="pure-group">
                       昵称：<input class="pure-input-1-4" name="comment-name" type="text"
                                 placeholder="昵称">
                       &nbsp;&nbsp;
                       邮箱：<input class="pure-input-1-3" name="comment-email" type="email"
                                 placeholder="留下邮箱，可以收到回复提醒哦～">
                   </fieldset>
                   <textarea class="comment-content" name="comment" onblur="hidden_replyform(this)"
                             placeholder="想对作者说点什么。。。"></textarea>
                   <div class="comment-div">
                       <span id="tip_comment">支持markdown格式</span>
                       <button class="pure-button" type="submit" href="#">发表评论</button>
                   </div>
               </form>

           </li>
       {% endfor %}
   </ul>
{% endfor %}
```

comment/models.py

```python
from django.db import models


class Comment(models.Model):
    name = models.CharField(max_length=20)
    email = models.EmailField(max_length=50)
    website = models.URLField(blank=True)
    text = models.TextField()
    created_time = models.DateTimeField(auto_now_add=True)

    post = models.ForeignKey('Post.Post', on_delete=True)  # 一篇文章有多个评论

    # 评论
    reply_to = models.IntegerField(verbose_name="回复的哪条评论", default=0)
    reply_name = models.CharField(verbose_name="回复的哪个人", max_length=50, default=None)
    reply_email = models.EmailField(verbose_name="被回复的邮箱", max_length=50, default='chuxiaoyi@chuxiaoyi@cn')
    root_to = models.IntegerField(verbose_name="回复的是哪个主评论", default=0)

    class Meta:
        ordering = ['-created_time']
```

## 异步任务

celery目前已经完成支持django了，原始文档[戳这里](http://docs.celeryproject.org/en/latest/django/first-steps-with-django.html)。我主要在评论区回复邮件和增加pv（页面访问量）使用了异步。

首先，在和项目同名的包下的创建`celery.py`\(与settings.py同级\)文件：

```python
from __future__ import absolute_import, unicode_literals
import os
from celery import Celery
from django.conf import settings

# set the default Django settings module for the 'celery' program.
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'MySite.settings')

app = Celery('MySite', broker='redis://127.0.0.1:6379/3')

# Using a string here means the worker doesn't have to serialize
# the configuration object to child processes.
# - namespace='CELERY' means all celery-related configuration keys
#   should have a `CELERY_` prefix.
app.config_from_object('django.conf:settings')

# Load task modules from all registered Django app configs.
app.autodiscover_tasks(lambda: settings.INSTALLED_APPS)


@app.task(bind=True)
def debug_task(self):
    print('Request: {0!r}'.format(self.request))
```

并且。在和celery.py同级的`__init__.py`中添加如下代码：

```python
# This will make sure the app is always imported when
# Django starts so that shared_task will use this app.
from .celery import app as celery_app

__all__ = ('celery_app',)
```

然后，在需要使用异步的功能所在app中添加`tasks.py`。在这里，我加在了`Post`和`comment`里：

Post/tasks.py

```python
from celery import shared_task
from .models import Post
from django.db.models import F

@shared_task
def add_pv(post_id):
    """
    增加页面访问量
    :return:
    """
    Post.objects.filter(id=post_id).update(views=F('views') + 1)
```

comment/tasks.py

```python
from celery import shared_task
from django.core.mail import send_mail
from django.conf import settings


@shared_task
def async_send_mail(email, text, post_pk):
    """
    异步发送信息，告知被回复者有人评论
    :param email:
    :param text:
    :return:
    """
    title = "邮箱发送test"
    msg = "收到回复：" + text + \
          "\n跳转到www.chuxiaoyi.cn/detail/post-{}/查看".format(post_pk)
    from_email = settings.EMAIL_HOST_USER
    recievers = [email, ]
    print(email)
    send_mail(title, msg, from_email, recievers, fail_silently=False)
```

写好之后，在views.py中直接调用即可

```python
async_send_mail.delay(comment.reply_email, comment.text, pk)
```

```python
add_pv.delay(pk)
```

## 部署：通过supervisor管理celery

安装supervisor

```
sudo apt-get install supervisor
```

添加celery配置，在`/etc/supervisor/conf.d`文件夹下，创建`celery.conf`:

```
[program:celery]

#启动命令入口
command=/root/.pyenv/shims/celery worker -A MySite --loglevel=info

# 命令程序所在目录
directory=/home/BlogWebsite

# 运行命令的用户名
user=root

autostart=true
autorestart=true
stdout_logfile=/var/log/celery/blog_comment_mail_worker.log
stderr_logfile=/var/log/celery/blog_comment_mail_worker.log
```

后台运行celery

```
sudo supervisorctl start celery

sudo supervisorctl stop celery

sudo supervisorctl restart celery
```
