---
title: flask + SQLAlchemy设置读写分离
date: 2019-12-04 10:35:18
tags: flask sqlalchemy flask sqlalchemy
categories: python
---

<!--more-->

## 参考文档

> <https://www.jb51.net/article/174365.htm>  
> <https://gist.github.com/trustrachel/6828122#file-routing-py>

## 步骤

在配置中添加以下配置

```python
SQLALCHEMY_DATABASE_URI = 'xxx'
SQLALCHEMY_BINDS = {
        'xxx',
        'xxx',
    }
```

改写session

```python
from flask_sqlalchemy import SQLAlchemy, get_state
import sqlalchemy.orm as orm
from functools import partial

import logging

log = logging.getLogger(__name__)


class AutoRouteSession(orm.Session):

    def __init__(self, db, autocommit=False, autoflush=False, **options):
        self.app = db.get_app()
        self._model_changes = {}
        orm.Session.__init__(self, autocommit=autocommit, autoflush=autoflush,
                             bind=db.engine,
                             binds=db.get_binds(self.app), **options)

    def get_bind(self, mapper=None, clause=None):
        """
        根据配置及读写操作，自动更改数据库引擎
        Args:
            mapper:
            clause:

        Returns:

        """
        try:
            state = get_state(self.app)
        except (AssertionError, AttributeError, TypeError) as err:
            log.info("获取配置失败，使用默认数据库:{}".format(err))
            return orm.Session.get_bind(self, mapper, clause)

        # 如果没有设置SQLALCHEMY_BINDS，则默认使用SQLALCHEMY_DATABASE_URI
        if state is None or not self.app.config['SQLALCHEMY_BINDS']:
            if not self.app.debug:
                log.debug("未获取数据库绑定信息(SQLALCHEMY_BINDS),使用默认数据库")
            return orm.Session.get_bind(self, mapper, clause)

        # insert、update、delete操作使用master
        elif self._flushing:
            log.debug("当前使用master")
            return state.db.get_engine(self.app, bind='master')

        # 其他操作使用slave
        else:
            log.debug("当前使用slave")
            return state.db.get_engine(self.app, bind='slave')


class AutoRouteSQLAlchemy(SQLAlchemy):

    def create_scoped_session(self, options=None):
        """
        用于工厂类创建session
        Args:
            options:

        Returns:

        """
        if options is None:
            options = {}
        scopefunc = options.pop('scopefunc', None)
        return orm.scoped_session(
            partial(AutoRouteSession, self, **options), scopefunc=scopefunc
        )
```

实例化

```python
db = AutoRouteSQLAlchemy()
```