---
title: Django使用MySQL后端日期不能按月过滤的问题及解决方案
date: 2018-09-30 21:37:34
tags: django
categories: python
---

<!--more-->

参考文档：  
[https://chowyi.com/Django\%E4\%BD\%BF\%E7\%94\%A8MySQL\%E5\%90\%8E\%E7\%AB\%AF\%E6\%97\%A5\%E6\%9C\%9F\%E4\%B8\%8D\%E8\%83\%BD\%E6\%8C\%89\%E6\%9C\%88\%E8\%BF\%87\%E6\%BB\%A4\%E7\%9A\%84\%E9\%97\%AE\%E9\%A2\%98\%E5\%8F\%8A\%E8\%A7\%A3\%E5\%86\%B3\%E6\%96\%B9\%E6\%A1\%88/](https://chowyi.com/Django%E4%BD%BF%E7%94%A8MySQL%E5%90%8E%E7%AB%AF%E6%97%A5%E6%9C%9F%E4%B8%8D%E8%83%BD%E6%8C%89%E6%9C%88%E8%BF%87%E6%BB%A4%E7%9A%84%E9%97%AE%E9%A2%98%E5%8F%8A%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88/)

在终端中，输入：

```shell
mysql_tzinfo_to_sql /usr/share/zoneinfo | mysql -u root mysql
```