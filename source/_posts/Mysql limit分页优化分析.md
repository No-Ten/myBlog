---
title: MySQL limit分页优化分析
data: 2023-09-11 15:25:00
comment: 'valine'
category: MySQL优化
tags:
  - MySQL优化
---



# MySQL limit分页优化分析

## 背景

在某一次数据处理的过程中，发现某一部分的处理逻辑特别慢（涉及到查询数据库、调用远程接口批量上传数据等），需要优化。

排查思路：

1.通过日志定位分析哪一部分比较耗时；

2.考虑是否需要更改多线程？

3.代码逻辑是否需要优化？

4.数据表结构是否需要优化？加索引？

5.查询语句是否需要优化？

在代码层面做相应的更改后，查询速度提升不明显（改为并发），所以从数据库层面入手。以下为查询语句。

```sql
select * from table 
where create_or_update_time 
between '2023-09-11 00:00:00' and '2023-09-12 00:00:00' 
limit 100,100;
```

通过这条语句已经对比数据表结构，发现这张表只有id是主键，没有其他索引，create_or_update_time这个字段也不是索引。所以在这个表添加create_or_update_time这个索引。经过查询对比，速度有提升了一点，但是还不是特别明显，需要继续排除。

通过输出的日志分析，随着limit n,m 中的n增大，查询速度也随着变慢，原因可能在于limit 分页查询？下面我们来研究一下limit的分页原理。

## limit 查询底层分析

1）limit语句的查询时间与起始记录（offset）的位置成正比 ；

2）mysql的limit语句是很方便，但是对记录很多:百万，千万级别的表并不适合直接使用。

为啥limit n,m会随着n的增大，查询速度会变慢呢，原因在于**limit n,m 分页原理是先查询前n+m条，再在丢弃前n条，返回m条的。**

例如： limit10000,20的意思扫描满足条件的10020行，扔掉前面的10000行，返回最后的20行，问题就在这里。  LIMIT 2000000, 30 扫描了200万+ 30行，怪不得慢的都堵死了，甚至会导致磁盘io 100%消耗。  但是: limit 30 这样的语句仅仅扫描30行。



## 优化方案

### 1.在查询条件中添加索引

```sql
select * from table
where create_or_update_time 
between '2023-09-11 00:00:00' and '2023-09-12 00:00:00' 
limit 100,100;
```

如，在这条语句中，添加create_or_update_time索引，可加快查询，但是还不够，当limit中的分页过大还是很慢，需要继续优化。

### 2.利用inner join

```sql
select * from table t1
inner join (
select id from table where create_or_update_time between '2023-09-11 00:00:00' and '2023-09-12 00:00:00' limit 100,100;
) t2 using(id);
```

这条查询语句是先根据条件检索符合条件的主键id，再根据id内连接去获取需要的数据。使用内连接进行关联查询，只关联有共同id的，也不会建立临时表，所以速度比较快。对比了一下，速度起码提升了10倍，数据量越大，提升越明显。



## 扩展文章

MySQL分页使用limit和offset参数，为什么会导致执行变慢？https://zhuanlan.zhihu.com/p/584474378

mysql的limit性能优化  http://www.taodudu.cc/news/show-338436.html



聚簇索引和非聚簇索引  https://mp.weixin.qq.com/s?__biz=MzAxMjY5NDU2Ng==&mid=2651861539&idx=1&sn=77e036f27b300dfddc979842f3255b30&chksm=8049776ab73efe7cdc3c777e6c499bfcd92538d97210e8b745e7402d2e1068367a9877f8dca4&scene=27