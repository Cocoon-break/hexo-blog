---
title: MySQL系列五之原理及性能
date: 2018-08-25 21:17:22
index_img:
- /images/mysql/logo-mysql.png
tags: 
- mysql
categories:
- DB
---

### 索引原理分析

MySQL的索引是由**存储引擎**来实现的。由于存储引擎不同，所以具有不同的索引类型，如BTree索引，B+Tree索引，哈希索引，全文索引等。MySQL的InnoDB引擎就是基于B+Tree索引的。

聚簇索引和非聚簇索引：https://www.cnblogs.com/rjzheng/p/9915754.html

B+tree

#### 索引分类

1. 主键索引：即主索引，根据主键pk_clolum（length）建立索引，**不允许重复，不允许空值**

   ```mysql
   ALTER TABLE 'table_name' ADD PRIMARY KEY pk_index('col')；
   ```

2. 唯一索引：用来建立索引的列的值必须是**唯一的，允许空值**

   ```mysql
   ALTER TABLE 'table_name' ADD UNIQUE index_name('col')；
   ```

3. 普通索引：用表中的普通列构建的索引，没有任何限制

   ```mysql
   ALTER TABLE 'table_name' ADD INDEX index_name('col')；
   ```

4. 全文索引：用大文本对象的列构建的索引

   ```mysql
   ALTER TABLE 'table_name' ADD FULLTEXT INDEX ft_index('col')；
   ```

5. 组合索引：用多个列组合构建的索引，这多个列中的值**不允许有空值**

   ```mysql
   ALTER TABLE 'table_name' ADD INDEX index_name('col1','col2','col3')；
   ```

   - 遵循“最左前缀”原则，把最常用作为检索或排序的列放在最左，依次递减，组合索引***\*相当于建立了col1,col1col2,col1col2col3三个索引\****，而col2或者col3是不能使用索引的。
   - 在使用组合索引的时候可能因为列名长度过长而导致索引的key太大，导致效率降低，在允许的情况下，可以只取col1和col2的前几个字符作为索引

#### 索引的使用

##### 什么时候使用索引

- 主键自动建立唯一索引

- 经常作为查询条件在WHERE或者ORDER BY 语句中出现的列要建立索引

- 作为排序的列要建立索引

- 查询中与其他表关联的字段，外键关系建立索引

- 高并发条件下倾向组合索引

- 用于聚合函数的列可以建立索引，例如使用了max(column_1)或者count(column_1)时的column_1就需要建立索引

  

##### 什么时候不要使用索引

- 经常增删改的列不要建立索引
- 有大量重复的列不建立索引
- 表记录太少不要建立索引

##### 如何使用索引

1. 查看表中的索引

   ```mysql
   SHOW INDEX FROM tablename
   ```

2. 查看查询语句使用索引的情况，查询语句加**explain**

   ```mysql
   explain SELECT * FROM table_name WHERE column_1='123';
   ```

3. 创建索引

   ```mysql
   -- 创建表时添加所有，index
   CREATE TABLE mytable(  
       ID INT NOT NULL,   
       username VARCHAR(16) NOT NULL,  
       INDEX [indexName] (username(length))  
   );
   -- 创建表之后添加索引
   ALTER TABLE my_table ADD [UNIQUE] INDEX index_name(column_name);
   -- 或者
   CREATE INDEX index_name ON my_table(column_name);
   ```

4. 删除索引

   ```mysql
   DROP INDEX my_index ON tablename;
   ALTER TABLE table_name DROP INDEX index_name;
   ```

5. 根据索引查询，以下为使用的一部分

   ```mysql
   SELECT * FROM table_name WHERE column_1=column_2;-- (为column_1建立了索引)
   SELECT * FROM table_name WHERE column_1 LIKE '三%'
   SELECT * FROM table_name WHERE column_1 LIKE '_好_'
   ```

##### 索引失效原因

- 在组合索引中不能有列的值为NULL，如果有，那么这一列对组合索引就是无效的。
- 在一个SELECT语句中，索引只能使用一次，如果在WHERE中使用了，那么在ORDER BY中就不要用了。
- LIKE操作中，'%aaa%'不会使用索引，也就是索引会失效，但是‘aaa%’可以使用索引。
- 在索引的列上使用表达式或者函数会使索引失效，例如：select * from users where YEAR(adddate)<2007
- 在查询条件中使用不等于，包括<符号、>符号和！=会导致索引失效。特别的是如果对主键索引使用！=则不会使索引失效，如果对主键索引或者整数类型的索引使用<符号或者>符号不会使索引失效。（不等于，包括&lt;符号、>符号和！，如果占总记录的比例很小的话，也不会失效）
- 在查询条件中使用IS NULL或者IS NOT NULL会导致索引失效。
- 字符串不加单引号会导致索引失效。更准确的说是类型不一致会导致失效，比如字段email是字符串类型的，使用WHERE email=99999 则会导致失败。
- 在查询条件中使用OR连接多个条件会导致索引失效，除非OR链接的每个条件都加上索引，这时应该改为两次查询，然后用UNION ALL连接起来
- 如果排序的字段使用了索引，那么select的字段也要是索引字段，否则索引失效。特别的是如果排序的是主键索引则select * 也不会导致索引失效。
- 尽量不要包括多列排序，如果一定要，最好为这队列构建组合索引



### 执行计划

简单来说，执行计划是Mysql执行一条sql 时的表现，包括SQL查询的顺序、是否使用索引、以及使用的索引信息等内容，通常用于SQL的性能分析、优化等场景。

基本语法是：SQL语句前面添加关键字 explain

```sql
explain select * from stu where id >=2 order by name
```

mysql 执行计划主要包含以下信息

| id   | select_type | table | partitions | type | possible_keys | key  | key_length | ref  | rows | extra |
| ---- | ----------- | ----- | ---------- | ---- | ------------- | ---- | ---------- | ---- | ---- | ----- |
|      |             |       |            |      |               |      |            |      |      |       |

- **id** 表示各个子查询执行的顺序

  - id 相同执行顺序由上至下
  - id 不同，id 值越大越先被执行

- **select_type** 查询数据的操作类型

  | select_type  | 说明                                   |
  | ------------ | -------------------------------------- |
  | SIMPLE       | 不包含任何子查询或union等查询          |
  | PRIMARY      | 包含子查询最外层查询就显示为 `PRIMARY` |
  | SUBQUERY     | 在`select`或 `where`字句中包含的查询   |
  | DERIVED      | `from`字句中包含的查询                 |
  | UNION        | 出现在`union`后的查询语句中            |
  | UNION RESULT | 从UNION中获取结果集                    |

- **table** 输出的行所引用的表

- **partitions** 如果查询是基于分区表的话，显示查询将访问的分区。

- **type** 表示访问类型，性能由高到底，system const 性能最高，ALL性能最低。一般来说，得保证查询至少达到range级别，最好能达到ref。

  | type            | 说明                                                       |
  | --------------- | ---------------------------------------------------------- |
  | system const    | 连接类型的特例，查询的表为系统表                           |
  | const           | 使用主键或者唯一索引，且匹配的结果只有一条记录             |
  | eq_ref          | 在`join`查询中使用`PRIMARY KEY`or`UNIQUE NOT NULL`索引关联 |
  | ref             | 使用非唯一索引查找数据                                     |
  | fulltext        | 使用全文索引                                               |
  | ref_or_null     | 对`Null`进行索引的优化的 ref                               |
  | unique_subquery | 在子查询中使用 eq_ref                                      |
  | index_subquery  | 在子查询中使用 ref                                         |
  | range           | 索引范围查找                                               |
  | index           | 遍历索引                                                   |
  | ALL             | 扫描全表数据                                               |

- **possible_keys** 指出MySQL能使用哪个索引在该表中找到行。查询涉及到的字段上若存在索引，则该索引将被列出，但不一定被查询使用。如果该值为 NULL，说明没有使用索引，可以建立索引提高性能。

- **key** 显示MySQL实际决定使用的键。如果没有索引被选择，键是NULL

- **key_length** 表示索引中使用的字节数，通过该列计算查询中使用的索引的长度。在不损失精确性的情况下，长度越短越好显示的是索引字段的最大长度，并非实际使用长度

- **ref** 显示该表的索引字段关联了哪张表的哪个字段

- **rows** 根据表统计信息及选用情况，大致估算出找到所需的记录或所需读取的行数，数值越小越好。

- **extra** 包含不适合在其他列中显示但十分重要的额外信息

  | extra           | 说明                                                         |
  | --------------- | ------------------------------------------------------------ |
  | Using filesort  | 表示 mysql 对结果集进行外部排序，不能通过索引顺序达到排序效果。一般有 using filesort都建议优化去掉，因为这样的查询 cpu 资源消耗大，延时大。 |
  | using temporary | 查询有使用临时表, 一般出现于排序， 分组和多表 join 的情况， 查询效率不高，建议优化。 |
  | using index     | 使用覆盖索引，避免了访问表的数据行。效率不错                 |
  | Using where     | sql使用了where过滤,效率较高                                  |



### 性能分析思路

#### 问题定位

1. 通过命令查看MySQL的状态，来找到系统瓶颈。以下都是已经登录mysql 之后使用的命令。

   ```shell
   # 显示状态信息,show status like ‘XXX’
   show status;
   #显示系统变量,show variables like ‘XXX’
   show variables;
   #显示InnoDB存储引擎的状态
   show engine innodb status;
   #查看当前SQL执行，包括执行状态、是否锁表等
   show processlist;
   ```

   以下为退出MySQL登录之后执行

   ```shell
   # 显示状态信息
   mysqladmin extended-status -u username -p password
   #显示系统变量
   mysqladmin variables -u root -p password
   ```

   常用的主要是show status和show processlist

2. 慢日志查询。慢查询日志可以帮助我们知道哪些SQL语句执行效率低下

   ```sql
   -- 检查是否开启
   show variables like '%slow%';
   -- 如果没有开启，也可以在运行时开启这个参数。说明是动态参数
   set global slow_query_log=ON;
   -- 设置慢查询记录查询耗时多长的SQL,这里演示用100毫秒
   set long_query_time = 0.1;
   ```

   日志文件在/var/lib/mysql/mysql目录下，如果不在查看下MySQL的配置文件。

3. explain分析查询。具体使用参考**执行计划**部分内容。

4. profiling分析查询。通过profiling命令得到更准确的SQL执行消耗系统资源的信息。profiling默认是关闭的。

   ```sql
   -- 查看是否开启profiling
   select @@profiling;
   -- 开profiling。注意测试完关闭该特性，否则耗费资源
   set profiling=1;
   -- 查看所有记录profile的SQL
   show profiles;
   -- 查看指定ID的SQL的详情
   show profile for query 1;
   -- 测试完，关闭该特性
   set profiling=0;
   ```

#### 解决问题

当MySQL发现性能问题。以下是一些优化思路。

1. 优化SQL语句。以下是一些SQL使用建议
   - 当结果集只有一行数据时使用LIMIT 1
   - 多用like、不用null和where
   - 在join的时候中结果集更小的部分join更大的部分，这样可以减少缓存的开销
   - 避免SELECT *，始终指定你需要的列。从表中读取越多的数据，查询会变得更慢。
   - 使用连接（JOIN）来代替子查询(Sub-Queries) : 连接（JOIN）.. 之所以更有效率一些，是因为MySQL不需要在内存中创建临时表来完成这个逻辑上的需要两个步骤的查询工作。
   - 用ENUM、CHAR 而不是VARCHAR，使用合理的字段属性长度
   - 尽可能的使用NOT NULL
   - 拆分大的DELETE 或INSERT 语句
   - 查询的列越小越快
2. 建立索引。索引建立参考上面部分。索引使用建议
   - 索引字段上不用mysql函数
   - 在= 、group by 和 order by字段上面加上索引
   - 一个表的索引数最好不要超过6个，若太多则应考虑一些不常使用到的列上建的索引是否有必要。
   - 在使用in的时候可以尝试使用exists试试
   - 在join的时候减少extra字段中临时表的数量
   - 越小的数据类型通常更好：越小的数据类型通常在磁盘、内存和CPU缓存中都需要更少的空间，处理起来更快。
   - 简单的数据类型更好：整型数据比起字符，处理开销更小，因为字符串的比较更复杂。在MySQL中，应该用内置的日期和时间数据类型，而不是用字符串来存储时间；以及用整型数据类型存储IP地址。



参考：https://blog.csdn.net/tongdanping/article/details/79878302

​			https://juejin.im/post/5a52386d51882573443c852a
​			https://kaimingwan.com/post/shu-ju-ku/mysqlxing-neng-fen-xi-fang-fa-gong-ju-jing-yan-zong-jie