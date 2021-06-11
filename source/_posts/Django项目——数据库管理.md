---
title: Django项目——数据库管理
date: 2019-03-21 12:12:18
tags: python django
categories: django
---



最近有一个需求。是可以通过可视化界面分别操作测试环境/生产环境的数据库。这里我使用了Django框架。在这里记录一下重点知识点>.\<

<!--more-->

具体需求是：

1.  可以查看/修改数据库
2.  不同的人员有不同的权限
3.  可以查看修改历史，方便回滚

我的实现方式是：

0.  开发环境：python3.7 + django2.1
1.  直接利用django的admin，以及内置权限进行操作
2.  django连接多数据库
3.  通过继承django的LogEntry编写History类，从而达到记录修改历史的作用

## 多数据库

因为需要分别操作线上和测试的数据库，所以我采用**多app对应多数据库**的方式。我们需要做以下工作（假设这里已经创建好了三个app）:

**需要在app的model中指定`app_label`:**

```python
class SourceTest(models.Model):
	············
    class Meta:
        ··············
        app_label = 'TestApp'	# 这个app_label会在Setting文件和路由中用到
```

**在`setting.py`文件中添加：**

```python

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'Default',
        'USER': 'xxxx',
        'PASSWORD': 'xxxx',
        "HOST": "127.0.0.1",
        "PORT": "3306",
    },
    'test': {
            'ENGINE': 'django.db.backends.mysql',
            'NAME': 'Test',
            'USER': 'xxxx',
            'PASSWORD': 'xxxx',
            "HOST": "127.0.0.1",
            "PORT": "3306",
        },
    'online': {
            'ENGINE': 'django.db.backends.mysql',
            'NAME': 'Online',
            'USER': 'xxxx',
            'PASSWORD': 'xxxx',
            "HOST": "127.0.0.1",
            "PORT": "3306",
        },
}

DATABASE_ROUTERS = ['SqlManageer.database_router.DatabaseAppsRouter']
DATABASE_APPS_MAPPING = {
    'TestApp': 'test',
    'OnlineApp': 'online',
}
```

通过`DATABASE_APPS_MAPPING`参数可以看出我的app对应的数据库。之前也提到过，要记录操作历史\(UserHistory\)，这里我没有指定UserHistory这个app对应的数据库，那么他会默认到default的数据库。这里这样设计的原因是：UserHistory需要用user\_id作为外键，Django目前不支持跨数据库外键，那么就需要将UserHistory和user放到同一个数据库。

**在与工程同名的文件夹下创建`database_router.py`:**

```python
from django.conf import settings

DATABASE_MAPPING = settings.DATABASE_APPS_MAPPING


class DatabaseAppsRouter(object):
    """
    A router to control all database operations on models for different
    databases.

    In case an app is not set in settings.DATABASE_APPS_MAPPING, the router
    will fallback to the `default` database.

    Settings example:

    DATABASE_APPS_MAPPING = {'app1': 'db1', 'app2': 'db2'}
    """

    def db_for_read(self, model, **hints):
        """"Point all read operations to the specific database."""
        if model._meta.app_label in DATABASE_MAPPING:
            return DATABASE_MAPPING[model._meta.app_label]
        return None

    def db_for_write(self, model, **hints):
        """Point all write operations to the specific database."""
        if model._meta.app_label in DATABASE_MAPPING:
            return DATABASE_MAPPING[model._meta.app_label]
        return None

    def allow_relation(self, obj1, obj2, **hints):
        """Allow any relation between apps that use the same database."""
        db_obj1 = DATABASE_MAPPING.get(obj1._meta.app_label)
        db_obj2 = DATABASE_MAPPING.get(obj2._meta.app_label)
        if db_obj1 and db_obj2:
            if db_obj1 == db_obj2:
                return True
            else:
                return False
        return None

    # Django 1.7 - Django 1.11
    def allow_migrate(self, db, app_label, model_name=None, **hints):
        print(db, app_label, model_name, hints)
        if db in DATABASE_MAPPING.values():
            return DATABASE_MAPPING.get(app_label) == db
        elif app_label in DATABASE_MAPPING:
            return False
        return None
```

**数据库迁移**  
我们可以通过指定app或数据库进行迁移：

```python
./manage.py makemigrations 	# 这个命令会对所有的数据进行预迁移

# 可以指定数据库进行迁移，不指定默认到default
./manage.py migrate --database=test
./manage.py migrate --database=online
# 也可以同时指定数据库和app
./manage.py migrate --database=test TestApp
./manage.py migrate --database=online OnlineApp
./manage.py migrate UserHistor
```

## History类编写

通过阅读源码（[django.contrib.admin.options.py](http://django.contrib.admin.options.py)）可以发现，当发生增删改操作的时候，django内部都会记录日志，具体的方法是就是下面这三个方法了。可以看出，django就是通过在发生相关操作时，将操作的信息记录到LogEntry模块中。

```python
class ModelAdmin(BaseModelAdmin):
	    def log_addition(self, request, object, message):
        """
        Log that an object has been successfully added.

        The default implementation creates an admin LogEntry object.
        """
        from django.contrib.admin.models import LogEntry, ADDITION
        return LogEntry.objects.log_action(
            user_id=request.user.pk,
            content_type_id=get_content_type_for_model(object).pk,
            object_id=object.pk,
            object_repr=str(object),
            action_flag=ADDITION,
            change_message=message,
        )

    def log_change(self, request, object, message):
        """
        Log that an object has been successfully changed.

        The default implementation creates an admin LogEntry object.
        """
        from django.contrib.admin.models import LogEntry, CHANGE
        return LogEntry.objects.log_action(
            user_id=request.user.pk,
            content_type_id=get_content_type_for_model(object).pk,
            object_id=object.pk,
            object_repr=str(object),
            action_flag=CHANGE,
            change_message=message,
        )

    def log_deletion(self, request, object, object_repr):
        """
        Log that an object will be deleted. Note that this method must be
        called before the deletion.

        The default implementation creates an admin LogEntry object.
        """
        from django.contrib.admin.models import LogEntry, DELETION
        return LogEntry.objects.log_action(
            user_id=request.user.pk,
            content_type_id=get_content_type_for_model(object).pk,
            object_id=object.pk,
            object_repr=object_repr,
            action_flag=DELETION,
        )
```

利用这一特点。我们可以知道，只需要app在发生增删改操作的时候，将操作信息记录到我们自己定义的History中就可以了。因此，我继承了`admin.ModelAdmin`,重写了相关操作:

```python
class NewBaseAdmin(admin.ModelAdmin):
    def log_addition(self, request, object, message):
        from django.contrib.admin.models import ADDITION
		
		# 下面四行是为了将修改后的结果也记录到History中
        req_k = request._post.dict().keys()
        model_k = [item.column.lower() for item in object._meta.fields]
        k = req_k & model_k
        model_field = {item: object.__dict__[item] for item in k}
		
        return History.objects.log_action(
            user_id=request.user.pk,
            content_type_id=get_content_type_for_model(object).pk,
            object_id=object.pk,
            object_repr=str(object),
            action_flag=ADDITION,
            change_message=message,
            **model_field
        )

    def log_change(self, request, object, message):
        """
        Log that an object has been successfully changed.

        The default implementation creates an admin LogEntry object.
        """
        from django.contrib.admin.models import CHANGE
        req_k = request._post.dict().keys()
        model_k = [item.column for item in object._meta.fields]
        k = req_k & model_k
        model_field = {item: object.__dict__[item] for item in k}

        action = History.objects.log_action(
            user_id=request.user.pk,
            content_type_id=get_content_type_for_model(object).pk,
            object_id=object.pk,
            object_repr=str(object),
            action_flag=CHANGE,
            change_message=message,
            **model_field
        )
        return action

    def log_deletion(self, request, object, object_repr):
        """
        Log that an object will be deleted. Note that this method must be
        called before the deletion.

        The default implementation creates an admin LogEntry object.
        """
        from django.contrib.admin.models import DELETION
        req_k = request._post.dict().keys()
        model_k = [item.column.lower() for item in object._meta.fields]
        k = req_k & model_k
        model_field = {item: object.__dict__[item] for item in k}
        return History.objects.log_action(
            user_id=request.user.pk,
            content_type_id=get_content_type_for_model(object).pk,
            object_id=object.pk,
            object_repr=object_repr,
            action_flag=DELETION,
            **model_field
        )
```

上述代码编写过后，我们就可以在admin中直接继承我们新写的`NewBaseAdmin`了。而我们发现，LogEntry的log\_action并没有记录我们History的字段：

```python
class LogEntryManager(models.Manager):
    use_in_migrations = True

    def log_action(self, user_id, content_type_id, object_id, object_repr, action_flag, change_message=''):
        if isinstance(change_message, list):
            change_message = json.dumps(change_message)
        return self.model.objects.create(
            user_id=user_id,
            content_type_id=content_type_id,
            object_id=str(object_id),
            object_repr=object_repr[:200],
            action_flag=action_flag,
            change_message=change_message,
        )
```

因此，我们需要重新定义History的manager，就像下面这样：

```python
class NewLogEntryManager(models.Manager):
    use_in_migrations = True

    def log_action(self, user_id, content_type_id, object_id, object_repr, action_flag, change_message='', **kwargs):
        if isinstance(change_message, list):
            change_message = json.dumps(change_message)
        return self.model.objects.create(
            user_id=user_id,
            content_type_id=content_type_id,
            object_id=str(object_id),
            object_repr=object_repr[:200],
            action_flag=action_flag,
            change_message=change_message,
            **kwargs
        )

class History(LogEntry):
	·······
	objects = NewLogEntryManager()
```

这样一来，我们就可以实现，当对app进行增删改操作的时候，就会将相关操作及一些我们需要的字段记录到History中了。

## 关于utime时间戳的问题

在定义模型时，有一个字段是:

```python
utime = models.DateTimeField('更新时间', auto_now=True)
```

在每一次操作过后，utime都会自动更新为本次操作的时间。但是查看utime发现，总是比本地时间少8小时，我的解决办法是：

```python
LANGUAGE_CODE = 'zh-hans'

TIME_ZONE = 'Asia/Shanghai'

USE_I18N = True

USE_L10N = True

USE_TZ = True
```

因为具体字段涉及了公司内部的字段，这里就不贴出源码了。哪里说的不清楚，希望大家多多指出～
