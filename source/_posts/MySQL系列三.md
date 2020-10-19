---
title: MySQL系列三之事务和锁
date: 2018-08-18 23:17:01
index_img:
- /images/mysql/logo-mysql.png
tags: 
- mysql
categories:
- DB
---

本篇来讲讲MySQL的事务以及锁

### 事务

事务是保证多个SQL操作的一致性，如果有一条SQL语句操作失败，则全部的SQL语句都失效。简单的说事务就是一组原子性的SQL语句。可以将这组语句理解成一个工作单元，要么全部执行要么都不执行。

至于支不支持事务是和MySQL的存储引擎相关的，我们这边使用的是MySQL8.0的版本，使用的默认引擎是InnoDB（MySQL5.5以后默认使用InnoDB存储引擎），而InnoDB是支持事务的。如果是其他版本的MySQL可以通过以下SQL语句查询。

```mysql
show engins;
```

#### 事务的特性

事务具有以下的特性

- 原子性

  事务中的所有操作要么全部提交成功，要么全部失败回滚。比如从取款机取钱，这个事务可以分成两个步骤:1划卡，2出钱。不可能划了卡，而钱却没出来，这两步必须同时完成，要么就不完成。

- 一致性

  数据库总是从一个一致性的状态转换到另一个一致性的状态。

- 隔离性

  一个事务所做的修改在提交之前对其它事务是不可见的。两个以上的事务不会出现交错执行的状态，因为这样可能会导致数据不一致。

- 持久性

  一旦事务提交，其所做的修改便会永久保存在数据库中。

Mysql的提交默认是自动提交，即发送一条sql执行一条。

#### 事务的提交

执行 `START TRANSACTION` 或 `BEGIN` 语句后，表示要开启一项事务处理。

- COMMIT 提交事务
- ROLLBACK 回滚事务

```mysql
START TRANSACTION;-- 或者begin
INSERT INTO stu (class_id,sname,sex)VALUES(2,'张帝','女');
COMMIT; -- rollback 回滚事务
```

#### 事务隔离

在上面我们说到事务具有隔离性，那么为什么事务需要隔离性呢？

当高并发访问会遇到多个事务的隔离问题，可能会出现以下问题：

1. 脏读：事务A读取了事务B更新的数据，然后B回滚操作，那么A读取到的数据是脏数据
2. 不可重复读：事务 A 多次读取同一数据，事务 B 在事务A多次读取的过程中，对数据作了更新并提交，导致事务A多次读取同一数据时，结果不一致。
3. 幻读：系统管理员A将数据库中所有学生的成绩从具体分数改为ABCDE等级，但是系统管理员B就在这个时候插入了一条具体分数的记录，当系统管理员A改结束后发现还有一条记录没有改过来，就好像发生了幻觉一样，这就叫幻读。

> 不可重复读的和幻读很容易混淆，不可重复读侧重于修改，幻读侧重于新增或删除。解决不可重复读的问题只需锁住满足条件的行，解决幻读需要锁表

MySQL拥有不同的事务隔离级别

| 事务隔离级别                 | 脏读 | 不可重复读 | 幻读 | 说明                                                         |
| ---------------------------- | ---- | ---------- | ---- | ------------------------------------------------------------ |
| 读未提交（read-uncommitted） | 是   | 是         | 是   | 最低的事务隔离级别，一个事务还没提交时，它做的变更就能被别的事务看到 |
| 不可重复读（read-committed） | 否   | 是         | 是   | 保证一个事物提交后才能被另外一个事务读取。另外一个事务不能读取该事物未提交的数据。 |
| 可重复读（repeatable-read）  | 否   | 否         | 是   | 多次读取同一范围的数据会返回第一次查询的快照，即使其他事务对该数据做了更新修改。事务在执行期间看到的数据前后必须是一致的。 |
| 串行化（serializable）       | 否   | 否         | 否   | 事务 100% 隔离，可避免脏读、不可重复读、幻读的发生。花费最高代价但最可靠的事务隔离级别。 |

#### 查询/设置MySQL的事务隔离级别

InoDB默认的事务隔离级别是repeatable-read

```mysql
-- 查询本次会话的事务隔离级别
SELECT @@SESSION.transaction_isolation
-- 查询全局的事务隔离级别
SELECT @@GLOBAL.transaction_isolation
-- 设置本次会话的事务隔离级别
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
-- 设置全局的事务隔离级别
SET GLOBAL TRANSACTION ISOLATION LEVEL READ COMMITTED;
```

### 锁机制

因为Mysql支持多线程方式，所以可以同时处理多个客户端请求。但是为了防止客户端同时修改数据，所以有了锁的机制。锁的等级主要分为以下几种

- 表级锁：开销小，加锁快；不会出现死锁；锁定粒度大，发生锁冲突的概率最高 ，并发度最低。

- 页面锁：开销和加锁时间界于表锁和行锁之间；会出现死锁；锁定粒度界于表锁和行锁之间，并发度一般。

- 行级锁：开销大，加锁慢；会出现死锁；锁定粒度最小，发生锁冲突的概率最低，并发度也最高。

`InnoDB` 是主流储存引擎并支持行级锁的，有更高的并发处理性能。

- 行锁开销大，锁表慢
- 行锁高并发下可并行处理，性能更高
- 行锁是针对索引加的锁，在通过索引检索时才会应用行锁，否则使用表锁
- 在事务执行过程中，随时都可以执行锁定，锁在执行 COMMIT或者ROLLBACK的时候释放

#### 行锁

- 使用**索引字段**为筛选条件，来复现行锁。

  事务A执行以下sql 但不提交，此时已经将id=1这条记录给加锁了。

  ```mysql
  -- 事务A
  BEGIN;
  UPDATE stu SET sname = 'hdcms' WHERE id=1;
  ```

  事务B也执行对id=1进行相同的更新操作，执行过程出现阻塞。如果事务执行对id=2，则可以操作成功，不会出现阻塞现象。

  ```mysql
  -- 事务B 阻塞
  BEGIN;
  UPDATE stu SET sname = 'hdcms2' WHERE id=1;
  commit
  -- 事务B id=2 操作成功
  BEGIN;
  UPDATE stu SET sname = 'hdcms2' WHERE id=2;
  commit
  ```

  当事务A提交之后，解锁id=1这行记录之后，事务B的操作才会继续执行

  ```mysql
  -- 事务A
  commit
  ```

- 使用**非索引字段**为筛选条件，则出现表锁。如果出现表锁，则该张表的update操作将无法操作。

  事务A执行以下代码，因为`sname`字段没有添加索引，造成锁定整个表

  ```mysql
  -- 事务A 没有commit，同时sname 字段没有添加索引
  BEGIN;
  UPDATE stu SET sname = 'hdcms' WHERE sname ='haha1';
  ```

  现在事务B更新任何一条记录都会造成阻塞，因为现在是表锁状态

  ```mysql
  -- 事务B 阻塞，因为stu 表锁
  BEGIN;
  update stu set sname = '小明' where id=1
  commit
  ```

- 简单查看MySQL行锁的争用情况

  过检查InnoDB_row_lock状态变量来分析系统上的行锁的争夺情况

  ```mysql
  show status like 'innodb_row_lock%';
  ```

  如果发现争用比较严重，如Innodb_row_lock_waits和Innodb_row_lock_time_avg的值比较高，还可以通过设置InnoDB Monitors来进一步观察发生锁冲突的表、数据行等，并分析锁争用的原因。

#### 区间锁（页面锁）

- 使用索引字段作为筛选条件

  事务A筛选时使用了范围区间

  ```mysql
  -- 事务A 没有commit，造成范围锁id为2 和 3的行被锁住
  BEGIN;
  UPDATE goods SET num=200 WHERE id>1 AND id<4; 
  ```

  事务B将不能修改表中的ID等于2的记录，但可以修改大于等于4 或者等于1的记录

  ```mysql
  -- 事务B id=2 无法操作，区间锁
  BEGIN;
  update goods set num =1 where id=2;
  commit;
  -- 事务B id =4 正常操作
  BEGIN;
  update goods set num =1 where id=4;
  commit;
  ```

#### 表锁

MySQL表级锁定的常见类型主要分为两种，一种是读锁，一种是写锁。

##### 读锁

为表设置读锁后，当前会话和其他会话都不可以修改数据。

会话A对表goods设置了读锁，将不能修改该表，也不能操作其他表

```mysql
LOCK TABLE goods READ;-- 加读锁
UPDATE goods SET num=300 WHERE id=1;
SELECT * FROM stu;
```

因为会话A对表`goods`设置了读锁，所以会话B也不能修改

```mysql
update goods set num=200 where id=1;-- 阻塞住
```

会话A解锁表后，其他会话又可以继续操作表了

```mysql
UNLOCK TABLES;
```

##### 写锁

为表设置了写锁后，当前会话可以修改，查询表，其他会话将无法操作。

会话A对表goods设置写锁，本会话可以正常操作表， 并不能操作其他表

```mysql
LOCK TABLE goods WRITE; -- 加写锁
INSERT INTO goods (name,num )VALUES('后盾人教程',300);
```

会话B读取/写入/写入表数据都将阻塞

```mysql
select * from goods;-- 阻塞
```

会话A解锁表数据后，其他会话都可以正常操作了

```mysql
UNLOCK TABLES;
```

##### 悲观锁

悲观锁指对数据被外界修改持保守态度，在整个数据处理过程中，将数据处于锁定状态，可以很好地解决并发事务的更新丢失问题。

事务A执行悲观锁操作后，其他事务执行将被阻塞

```mysql
-- 没有commit, for update 表示执行悲观锁
BEGIN;
SELECT * FROM goods WHERE id=1 FOR UPDATE;
UPDATE goods SET num=num-2 WHERE id=1; 
```

事务B执行以下代码将不能查询库存，必须等事务A提交或回滚事务

```mysql
-- 被阻塞，只要等事务A commit之后才会执行B事务
BEGIN;
SELECT * FROM goods WHERE id=1 FOR UPDATE;
commit；
```

##### 乐观锁

在每次去拿数据的时候认为别人不会修改，不对数据上锁，但是在提交更新的时候会判断在此期间数据是否被更改，如果被更改则提交失败。

事务A查询商品库存，获取了商品记录，记录中有VERSION字段用于记录版本号（目前为0）

```mysql
BEGIN;
SELECT * FROM goods WHERE id = 1;
```

事务B同时查询，也获取了版本号为0的记录

```mysql
BEGIN;
SELECT * FROM goods WHERE id = 1;
```

事务A更改库存，并增加版本号

```mysql
UPDATE goods SET num=num-10,VERSION =VERSION+1 WHERE VERSION=0;
```

事务B更改数据，但使用的是事务B查询到的0号版本，因为事务A已经提交版本号为1，造成事务B修改失败，保证了数据的完整性。

```mysql
UPDATE goods SET num=num-10,VERSION =VERSION+1 WHERE VERSION=0;
```

[超详细的锁介绍](https://blog.csdn.net/Jack__Frost/article/details/73347688)

在对数据进行insert\update\delete的时候容易出现Deadlock found when trying to get lock 错误，目前有两个方式进行调整优化。

1. 优化sql语句
2. 降低MySQL数据的事务隔离级别



参考链接：https://www.cnblogs.com/qq1148932219/p/11694064.html