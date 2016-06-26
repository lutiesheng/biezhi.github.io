---
layout: post
title: 对象缓存和n+1问题分析
categories: 架构
tags: 架构 缓存
---

* content
{:toc}

我们常见的web应用，性能瓶颈往往是数据库查询，因为应用服务器层面可以水平扩展，但是数据库是单点的，很难水平扩展，当数据库服务器发生磁盘IO，往往无法有效提高性能，因此如何有效降低数据库查询频率，减轻数据库磁盘IO压力，是web应用性能问题的根源。

对象缓存是所有缓存技术当中适用场景最广泛的，任何web应用，即使实时性要求很高，你也可以使用对象缓存，而且好的ORM实现，对象缓存是完全透明的，完全不需要你的程序代码进行硬编码。



用不用对象缓存，怎么用对象缓存，不是一个简单的代码调优技巧，而是整个应用的架构问题。在你开发一个应用之前，你就要想清楚，这个应用最终的场景是什么？会有多大的用户量和数据量。你将采用什么方式来架构这个应用：

也许你偏好对SQL语句级别的优化，数据库设计当大表有很多冗余字段，会尽量消除大表之间的关联关系，当数据量很大以后，选择分库分表的优化方式，这是目前业界常规做法。但是也可以选择使用ORM的对象缓存优化方式：数据库设计避免出现大表，比较多的表关联关系，通过ORM以对象化方式操作，利用对象缓存提升性能。举个例子：

论坛的列表页面，需要显示topic的分页列表，topic作者的名字，topic最后回复帖子的作者，常规做法：

```sql
select ... from topic left join user left join post .....
```
你需要通过join user表来取得topic作者的名字，然后你还需要join post表取得最后回复的帖子，post再join user表取得最后回贴作者名字。也许你说，我可以设计表冗余，在topic里面增加username，在post里面增加username，所以通过大表冗余字段，消除了复杂的表关联：

```sql
select ... from topic left join post...
```
OK，且不说冗余字段的维护问题，现在仍然是两张大表的关联查询。然后让我们看看ORM怎么做？

select * from topic where ... --分页条件
就这么一条SQL搞定，比上面的关联查询对数据库的压力小多了。 也许你说，不对阿，作者信息呢？回贴作者信息呢？这些难道不会发送SQL吗？如果发送SQL，这不就是臭名昭著的n+1条问题吗？ 你说的对，最坏情况下，会有很多条SQL：

```sql
select * from user where id = topic_id...;
....
select * from user where id = topic_id...;

select * from post where id = last_topic_id...;
....
select * from post where id = last_topic_id...;

select * from user where id = post_id...;
....
select * from user where id = post_id...;
```
事实上何止n+1，根本就是3n+1条SQL了。那你怎么还说ORM性能高呢？ 因为对象缓存在起作用，你可以观察到后面的3n条SQL语句全部都是基于主键的单表查询，这3n条语句在理想状况下(比较繁忙的web网站的热点数据)，全部都可以命中缓存。所以事实上只有一条SQL，就是：

```sql
select * from topic where ...--分页条件 
```
这条单表的条件查询和直接使用join查询SQL通过字段冗余简化过后的大表关联查询相比，当数据量大到一定程度以后对数据库磁盘IO的压力很小，这就是对象缓存的真正威力！

更进一步分析，使用ORM，我们不考虑缓存的情况，那么就是3n+1条SQL。但是这3n+1条SQL的执行速度一定比SQL的大表关联查询慢吗？不一定！因为使用ORM的情况下，第一条SQL是单表的条件查询，在有索引的情况下，速度很快，后面的3n条SQL都是单表的主键查询，在繁忙的数据库系统当中，3n条SQL几乎可以全部命中数据库的data buffer。但是使用SQL的大表关联查询，很可能会造成大范围的表扫描，造成频繁的数据库服务器磁盘IO，性能有可能是非常差的。

因此，即使不使用对象缓存，ORM的n+1条SQL性能仍然很有可能超过SQL的大表关联查询，而且对数据库磁盘IO造成的压力要小很多。这个结论貌似令人难以置信，但经过我的实践证明，就是事实。前提是数据量和访问量都要比较大，否则看不出来这种效果。

## 对象缓存的命中率

### 应用场景

即使是web应用，也要看访问的频度，一个极少被访问到的缓存等于没有什么效果。一般来说，互联网网站是非常适合缓存应用的场景。

### 缓存的粒度

毫无疑问，缓存的粒度越小，命中率就越高，对象缓存是目前缓存粒度最小的，因此被命中的几率更高。举个例子来说吧：你访问当前这个页面，浏览帖子，那么对于ORM来说，需要发送n条SQL，取各自帖子user的对象。很显然，如果这个user在其他帖子里面也跟贴了，那么在访问那个帖子的时候，就可以直接从缓存里面取这个user对象了。

### 架构的设计

架构的设计对于缓存命中率也有至关重要的影响。例如你应该如何去尽量避免缓存失效的问题，如何尽量提供频繁访问数据的缓存问题，这些都是考验架构师水平的地方。再举个例子来说，对于论坛，需要记录每个topic的浏览次数，所以每次有人访问这个topic，那么topic表就要update一次，这意味着什么呢？对于topic的对象缓存是无效的，每次访问都要更新缓存。那么可以想一些办法，例如增加一个中间变量记录点击次数，每累计一定的点击，才更新一次数据库，从而减低缓存失效的频率。

### 缓存的容量和缓存的有效期

缓存太小，造成频繁的LRU，也会降低命中率，缓存的有效期太短也会造成缓存命中率下降。

所以缓存命中率问题不能一概而论，一定说命中率很低或者命中率很高。但是如果你对于缓存的掌握很精通，有意识的去调整应用的架构，去分解缓存的粒度，总是会带来很高的命中率的。