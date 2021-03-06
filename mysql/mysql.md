# mysql基础知识

> 作者： 老了的程序员 Mr-Gaosai

### mysql的常见引擎

mysql常见引擎有：MyISAM和InnoDb

### MyISAM简介

MyISAM引擎不支持事务操作，因此没有事务的ACID和事务隔离技术，相对Innodb而言简单。
MyISAM不支持外键约束。
MyISAM锁的粒度为表级锁，因此当insert和update数据时，并发性效率会低一些。
MyIASM中存储了表的行数，于是SELECT COUNT(*) FROM TABLE时只需要直接读取已经保存好的值而不需要进行全表扫描。
MYISAM支持fulltext索引结构。
MYISAM适用于对表查询较多，插入修改少并且不需要事务处理的情境下。

### InnoDB简介

Innodb支持事务操作，并且实现了事务的隔离技术。
Innodb支持外键约束。
Innodb锁的粒度为行级锁，因此Insert和Update操作并发性能更好。
Innodb没有保存表中的行数，因此select count(*) from table 会统计整张表。
Innodb不支持fulltext索引结构。
InnoDB适用于事务处理，具有ACID事务支持等特性，在应用中执行大量insert和update操作，应该选择InnoDB

### Innodb与MyISAM的区别

1. 事务
2. 外键
3. count(*)
4. 锁的粒度
5. fulltext索引结构
6. B+Tree的结构（设计聚集索引和非聚集索引知识，下文解释）

### mysql索引简介

索引是帮助MySQL高效获取数据的数据结构。换句话说索引是数据结构。

### mysql索引类型

1. 单值索引
2. 唯一索引
3. 复合索引

### 聚集索引和非聚集索引的区别

聚集索引：该索引中键值的逻辑顺序决定了表中相应行的物理顺序

非聚集索引：该索引中索引的逻辑顺序与磁盘上行的物理存储顺序不同

### mysql索引结构

1. B-Tree
2. fulltext
3. hash
4. R-Tree

### 哪些情况需要创建索引

1. 频繁作为查询条件的字段
2. 主键默认添加唯一索引
3. 外键需要建立索引
4. order by的字段建立索引会提高速度
5. ...

### 哪些情况不需要创建索引

1. 表记录少
2. 频繁增删改的表（如果建立索引的话虽然提高的查找速度，但是插入等操作时需要维持索引的结构，带来了性能损耗）
3. 重复率高的字段（如果建立索引，带不来实际效果，如性别等字段）


### 什么情况下设置了索引但无法使用(SQL语句的优化)

* 全值匹配
* 最佳左前缀
* 索引列上做任何操作(计算,函数,类型转换等)
* 范围之后全失效
* 尽量少用*,而使用覆盖索引(只访问索引的查询)
* 在使用!=或者<>时,无法使用索引会导致全表扫描.
* is null,is not null无法使用索引
* like以通配符开头(‘%’)索引会失效.比如like(‘%lisi’)索引失效
* 字符串不加单引号导致索引失效
* 少用or,用or的时候也会导致索引失效.

### 最左索引原则

### B-Tree数据结构介绍

一颗多路复用的树，非搜索二叉树和进化了的红黑树。

每个节点保存了数据

### B+Tree数据结构介绍

一颗多路复用的树，非搜索二叉树和进化了的红黑树。

叶子节点保存了数据

### mysql采用的索引数据结构

MyISAM引擎采用了B+Tree的数据结构，但是叶子节点上保存的数据为数据的地址。索引和数据不在同一个文件中。
索引的逻辑顺序非数据的物理顺序，因此MyISAM的主键索引为非聚集索引

Innodb引擎采用了B+Tree的数据结构，但是叶子节点上保存的数据为表实际存储的数据。主键索引和数据位于同一个文件中。
索引的逻辑顺序为数据的物理顺序，因此Innodb的主键索引为聚集索引。
Innodb中的其他索引在不同于主键索引的文件中，当其他索引也采用B+Tree的结构，只不过叶子节点保存的是主键信息。
因此要获取数据还需要在主键索引中进行一次查询（总共两次查询）。

### InnoDB的主键选择与优化

在Innodb我们通过上面可以知道其他索引中保存了主键索引，因此主键索引长度小的话，其他索引文件的大小也会变小。

在Innodb中支持自增的主键。我们来讨论下自增索引的好处。
当插入数据时如果自增主键索引，那么他在主键索引文件中肯定位于最后一个位置。
而如果随机的主键插入时位置不确定我们还需要维持一个B+Tree的平衡问题，因此性能不急主键自增索引。

因此：在使用InnoDB存储引擎时，如果没有特别的需要，请永远使用一个与业务无关的自增字段作为主键。

> [索引基础知识](https://www.cnblogs.com/zhaobingqing/p/7066112.html)
> 
> [索引结构知识](https://www.cnblogs.com/tgycoder/p/5410057.html)
> 
> [引擎知识]    (http://blog.csdn.net/ls5718/article/details/52248040)
