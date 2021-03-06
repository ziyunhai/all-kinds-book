[toc]
## 物化
- 子查询语句中的子查询结果集中的记录保存到临时表的过程称之为 物化（英文名：Materialize），简单理解为存储子查询结果集的临时表称之为 物化表。

也正因为物化表的记录都建立了索引（基于内存的物化表有哈希索引，基于磁盘的有B+树索引），因此通过 IN 语句判断某个操作数在不在子查询的结果集中变得很快，从而提升语句的性能

## 半连接 semi-join
```
SELECT ... FROM outer_tables
    WHERE expr IN (SELECT ... FROM inner_tables ...) AND ...
```
- 对于outer_tables的某条记录来说，我们仅关心在inner_tables表中是否存在匹配的记录，而不用关心具体有多少条记录与之匹配，最终结果只保留 outer_tables 表的记录。

## 数据库三范式：
### 第一范式(属性原子性)
1NF是对属性的原子性约束，要求字段具有原子性，不可再分解；(只要是关系型数据库都满足1NF)
### 第二范式(部分依赖)
- 2NF是在满足第一范式的前提下，非主键字段不能出现部分依赖主键；解决：消除复合主键就可避免出现部分以来，可增加单列关键字。
### 第三范式(传递依赖)
- 3NF是在满足第二范式的前提下，非主键字段不能出现传递依赖，比如某个字段a依赖于主键，而一些字段依赖字段a，这就是传递依赖。解决：将一个实体信息的数据放在一个表内实现。

## 区分度/基数
- 个索引上不同的值的个数，我们称之为“基数”（cardinality）
- 基数越大，索引的区分度越好
### 采样统计基数
- InnoDB 默认会选择 N 个数据页，统计这些页面上的不同值，得到一个平均值，然后乘以这个索引的页面数，就得到了这个索引的基数。
- 数据表是会持续更新的，索引统计信息也不会固定不变。所以，当变更的数据行数超过 1/M 的时候，会自动触发重新做一次索引统计。
- 当统计信息不对，采用 analyze table t 命令，可以用来重新统计索引信息

### 存储索引统计的方式
- 可以通过设置参数 innodb_stats_persistent 的值来选择：
- 设置为 on 的时候，表示统计信息会持久化存储。这时，默认的 N 是 20，M 是 10。
- 设置为 off 的时候，表示统计信息只存储在内存中。这时，默认的 N 是 8，M 是 16

## Index Condition Pushdown (ICP)