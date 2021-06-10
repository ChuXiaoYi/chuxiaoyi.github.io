---
title: mysql innodb索引结构及相关原理
date: 2019-12-30 14:32:44
tags: 数据库 mysql
categories: myql
---

<!--more-->

> 最近在优化线上代码，优化过程中，对数据库进行了一系列的学习和基础巩固，看了很多大佬写的文章，整理了一下，记录在这里～

参考文档：

- [清空认知，然后重新理解MySQL索引结构](https://juejin.im/post/5def29f2518825121f6994f7#heading-2)
- [MySQL索引背后的数据结构及算法原理](http://blog.codinglabs.org/articles/theory-of-mysql-index.html)

## B+tree的结构

### 页

在操作系统中，当我们往磁盘中取数据的时候，如果我们想要取出1kb的数据时，会发现，操作系统取出了4kb的数据，这是因为操作系统中`页`的大小是4kb。这是为什么呢？

当一个程序访问了一条数据之后，很有可能在此访问这条数据或访问这条数据相邻的数据。因此，干脆直接把4kb的数据全部取出，放到内存中，下次再来访问时，直接从内存中取出，减少了磁盘IO。

假设，我们现在有一张用户表，我们要找出id=5的数据，在没了解页这个概念之前，最原始的方法就是`遍历`。我们会不断的从磁盘中取出一条数据，然后判断这条数据是不是id=5，如果不等于，会继续向后遍历，此时，我们会查找5次，也就是经过了5次磁盘IO。

现在，我们将页引入，当我们取出id=1的数据时，操作系统会将id=1所在页的全部数据\(id=1到id=4\)取出，放到内存中，我们继续向后遍历，接下来的遍历，会从内存中取出；当取出id=5的数据时，操作系统会再次将该id所在页全部取出。此时我们会发现，只经过了2次磁盘IO

> 在MySQL的InnoDb引擎中，页的大小是16KB，是操作系统的4倍，而int类型的数据是4个字节，其它类型的数据的字节数通常也在4000字节以内，所以一页是可以存放很多很多条数据的，而MySQL的数据正是以`页`为基本单位组合而成的。

页的结构如下：  
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019122716195320.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDE1NjQ4Nw==,size_16,color_FFFFFF,t_70)

### 页目录

> 通过对上面页的了解，我们在查找数据的时候，已经可以做到减少磁盘IO了。但是我们可以发现其实页中的数据结构就是一个`链表`。**链表的优点是插入和删除很快，但是查询需要按指针方向去遍历**。假设现在一页中有上百万条数据，最坏的情况他可能会遍历上百万次，即使在内存中，效率也不高。

页目录就像书目录一样，会告诉我们哪个标题从哪一页开始。页目录的结构如下：  
![[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-lag5GLOy-1577436044948)(evernotecid://FBE381A3-17C7-41D9-AA37-9C5F29FAB396/appyinxiangcom/20545635/ENResource/p158)]](https://img-blog.csdnimg.cn/20191227164059671.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDE1NjQ4Nw==,size_16,color_FFFFFF,t_70)

假设我们查找id=3的数据，首先会找到目录2，然后对目录2中的数据进行查找。当然我们能通过目录快速的找到数据，其实主要基于这些数据已经经过了排序。因此，可以得知，**在我们插入数据时，数据库会对数据进行排序。**

当然，不可能只有一页数据，多页数据的结构如下：![在这里插入图片描述](https://img-blog.csdnimg.cn/20191227171413247.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDE1NjQ4Nw==,size_16,color_FFFFFF,t_70)  
但是需要注意的是，**在开辟新页的时候，数据并不一定在新开辟的页上，而是需要进行和所有页的数据的比较，从而来决定数据存放的位置**。

### 目录页

> 从上图我们可以发现，多页之间也是通过指针相连，他们其实就是链表。如果页的数量过多，又会造成遍历过多，此时，我可以想到，有没有一个结构，可以像页目录那样，可以优化页内数据呢？

页内数据和多页他们本质上都是链表，所以我们可以采用相同的方式优化，这种方式叫目录页。  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191227182639980.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDE1NjQ4Nw==,size_16,color_FFFFFF,t_70)  
需要注意的是，**目录页的本质也是页，但是，普通页中村的数据是项目数据，而目录页中村的数据是普通页的地址。**

### B+tree

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191230114721619.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDE1NjQ4Nw==,size_16,color_FFFFFF,t_70)  
从上面的图我们已经可以看出，这其实就是b+tree的结构了。一般来说，树的深度是3层；每个叶子就是数据页，非叶子结点就是目录页；叶子结点存放真实的数据，而非叶子结点存放目录，也就是索引。

它的优势有以下几点：

1.  由于叶子结点上存放了所有的数据，并且有指针相连，每个叶子结点在逻辑上是相连的，所以对范围查找比较友好；
2.  B+tree所有的数据都在叶子结点上，所以查询效率稳定，一般都是查询三次；
3.  有利于数据库的扫描；
4.  有利于磁盘的IO，因为他的层高不会因为数据扩大而增高（三层树高大概可以存放2000万的数据量）；

## InnoDB索引

我们都知道innodb索引的结构是B+tree，那么，他是怎样存放数据的呢？  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191230142910144.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDE1NjQ4Nw==,size_16,color_FFFFFF,t_70)  
通过上面的图，我们可以得到以下信息：innodb的数据文件本身就是按B+tree组织的索引结构，**叶子结点存放了表的完整数据**，key是数据表的主键，因此，**innodb数据表文件本身就是主索引**。

> 像这种叶子结点包含完整数据的结构，我们叫它`聚簇索引`。聚簇索引是按照每张表的主键构造的一颗B+树，树的叶子结点存放着表中行数据。由于每张表的主键只有一个，因此**数据库中每张表只有一个聚簇索引。**  
> 因为InnoDB的数据文件本身要按主键聚集，所以InnoDB要求表必须有主键（MyISAM可以没有），如果没有显式指定，则MySQL系统会自动选择一个可以唯一标识数据记录的列作为主键，如果不存在这种列，则MySQL自动为InnoDB表生成一个隐含字段作为主键，这个字段长度为6个字节，类型为长整形。

除了主索引，还有辅助索引。每张表的辅助索引可以有多个，它的叶子结点不同于主索引，辅助索引存储着相应记录主键的值。也就是说，innodb的所有辅助索引都引用主键作为data。  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191230142849309.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDE1NjQ4Nw==,size_16,color_FFFFFF,t_70)

> 辅助索引也可以叫做`非聚簇索引`。非聚簇索引就是将数据和索引分开，查找时需要先查找到索引，然后通过索引回表找到相应的数据。

**因为所有辅助索引都引用主索引，过长的主索引会令辅助索引变得过大，所以不建议使用过长的字段作为主键。**