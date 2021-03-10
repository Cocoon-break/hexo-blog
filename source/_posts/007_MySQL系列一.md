---
title: MySQL系列一之基础使用
date: 2018-08-12 19:47:00
index_img:
- /images/mysql/007_mysql_0.jpg
tags: 
- mysql
categories:
- DB
---

MySQL是平常使用较多的数据，因项目需要发现自身对MySQL有所欠缺。所以复习了一下MySQL知识，做个总结归纳。本文介绍为基础知识，从数据库链接开始，包括一些常用SQL语句，和MySQL的一些数据类型。建议使用docker 的MySQL 进行SQL练习。同时推荐个人比较喜欢使用的数据库可视化软件Table Plus。

### 链接MySQL服务

通过命令行建立链接

```shell
# -u 参数指定用户名，-p指定密码，-P指定端口默认是3306，-h 是指定主机地址,-e 指定链接之后操作
mysql -uroot -proot -P3306 -h 127.0.0.1
mysql -uroot -proot -e"show databases" 
# 建立链接的同时，执行提前写好的sql文件
mysql -uroot -proot < test.sql
```

### 数据库管理

数据库管理包括，数据库创建，数据库删除，数据库使用等操作

```mysql
-- 创建数据库并指定字符集
create database test charset utf8;
-- 数据库查看
show databases;
-- 删除数据库
drop database test;
-- 一般为了删除不存在的数据库报错，可以通过if exits
drop database if exits test;
-- 使用数据库
use test;
```

### 数据表管理

数据表管理包括，数据表创建，数据表删除，数据表记录增删改，数据表字段名修改

```mysql
-- 创建class 表id 自增，同时指定表字符集
create table class (id int primary key AUTO_INCREMENT,cname varchar(30) NOT NULL,description varchar(100) default NULL) charset utf8;
-- 数据表插入单条数据，和多条数据
INSERT INTO class set cname ='golang',description='开发语言';
INSERT INTO class (cname,description) VALUES('PHP','开发语言'),('Mysql','数据库');
-- 数据表删除单条记录
delete from class where cname='golang';
-- 数据表修改单条记录
update class set cname = "PHP2" where id=2;
-- 删除表
drop table if exits class;
```

创建数据模板表，后续可以根据模板表创建新的数据表

```mysql
-- 复制表结构
create table tcopy like class;
-- 复制表同时复制数据
create table tcopy select * from class;
```

数据表字段名称修改以及字段增删

```mysql
-- 数据表重命名
alter table class rename classes;
rename table classes to class;
-- 数据表增加字段
alter table class add school varchar(50);
-- 数据表删除字段
alter table class drop school;
-- 修改字段名
alter table class CHANGE description descriptions varchar(30);
```

数据表相关主键操作，主键的增删

```mysql
-- 主键为自增字段，需要删除自增属性后才可以删除主键
alter table class MODIFY id int not null;-- 删除自增
alter table class DROP PRIMARY key;-- 删除主键

-- 添加表主键
alter table class add PRIMARY KEY(id);-- 添加表主键
alter table class MODIFY id int not null AUTO_INCREMENT;-- 添加自增列
alter table class modify id int not null AUTO_INCREMENT ,add PRIMARY key(id);
```

### 数据类型

MySQL数据类型包括字符串，数值类型，枚举类型

**字符串数据类型**

| 类型       | 大小                | 用途                            |
| :--------- | :------------------ | :------------------------------ |
| CHAR       | 0-255字节           | 定长字符串                      |
| VARCHAR    | 0-65535 字节        | 变长字符串                      |
| TINYBLOB   | 0-255字节           | 不超过 255 个字符的二进制字符串 |
| TINYTEXT   | 0-255字节           | 短文本字符串                    |
| BLOB       | 0-65 535字节        | 二进制形式的长文本数据          |
| TEXT       | 0-65 535字节        | 长文本数据                      |
| MEDIUMBLOB | 0-16 777 215字节    | 二进制形式的中等长度文本数据    |
| MEDIUMTEXT | 0-16 777 215字节    | 中等长度文本数据                |
| LONGBLOB   | 0-4 294 967 295字节 | 二进制形式的极大文本数据        |
| LONGTEXT   | 0-4 294 967 295字节 | 极大文本数据                    |

CHAR类型是定长的数据类型，比如定义20长度的`char`类型即使只存一个字符，也占20个长度，好处是处理速度快，缺点是占用空间大。

VARCHAR类型是变长数据类型，空间受内容长度影响。

**字符串常用函数**

| 函数名          | 作用                         | 示例                                                  |
| --------------- | ---------------------------- | ----------------------------------------------------- |
| UPPER()/LOWER() | 将内容全部改成大写/小写      | select UPPER(cname) from class;                       |
| CONCAT()        | 字符拼接                     | select concat(cname,description) from class;          |
| Left()/Right()  | 用于取左或右指定数量的字符   | select left(cname,3) from class;                      |
| mid             | 从中间取字符串               | select *  from class where mid(cname,2,2) = 'hp';     |
| substring       | 从指定位置开始向右截取字符串 | select *  from class where SUBSTRING(cname,2) = 'hp'; |
| char_length     | 获取字符串数量               | select char_length(cname) from class;                 |

正则表达式和like的使用

```mysql
-- 正则匹配php 或者MySQL
SELECT * FROM class WHERE cname REGEXP 'php|mysql';
-- _用于匹配一个字符，%用于匹配任意多个字符
SELECT *  FROM class WHERE cname LIKE '_h%';
```

**数值整型类型**

| MySQL数据类型 | 含义（有符号）                       |
| ------------- | ------------------------------------ |
| tinyint(m)    | 1个字节 范围(-128~127)               |
| smallint(m)   | 2个字节 范围(-32768~32767)           |
| mediumint(m)  | 3个字节 范围(-8388608~8388607)       |
| int(m)        | 4个字节 范围(-2147483648~2147483647) |
| bigint(m)     | 8个字节 范围(+-9.22*10的18次方)      |

取值范围如果加了unsigned，则最大值翻倍，如tinyint unsigned的取值范围为(0~256)。

**数值浮点型**

| 类型    | 大小                               | 范围（有符号）                                               | 范围（无符号）                                               |
| :------ | :--------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| FLOAT   | 4 字节                             | (-3.402 823 466 E+38，-1.175 494 351 E-38)，0，(1.175 494 351 E-38，3.402 823 466 351 E+38) | 0，(1.175 494 351 E-38，3.402 823 466 E+38)                  |
| DOUBLE  | 8 字节                             | (-1.797 693 134 862 315 7 E+308，-2.225 073 858 507 201 4 E-308)，0，(2.225 073 858 507 201 4 E-308，1.797 693 134 862 315 7 E+308) | 0，(2.225 073 858 507 201 4 E-308，1.797 693 134 862 315 7 E+308) |
| DECIMAL | DECIMAL(M,D) ，m<65 是总个数，d<30 | 依赖于M和D的值                                               | 依赖于M和D的值                                               |

**ENUM/SET**

ENUM 类型因为只允许在集合中取得一个值，有点类似于单选项。

SET 类型与 ENUM 类型相似但不相同。SET 类型可以从预定义的集合中取得任意数量的值。　一个 SET 类型最多可以包含 64 项元素。

### 日期时间

| 日期时间类型 | 占用空间 | 日期格式            | 最小值              | 最大值              | 零值表示            |
| ------------ | -------- | ------------------- | ------------------- | ------------------- | ------------------- |
| DATETIME     | 8 bytes  | YYYY-MM-DD HH:MM:SS | 1000-01-01 00:00:00 | 9999-12-31 23:59:59 | 0000-00-00 00:00:00 |
| TIMESTAMP    | 4 bytes  | YYYY-MM-DD HH:MM:SS | 1970-01-01 08:00:01 | 2038-01-19 03:14:07 | 00000000000000      |
| DATE         | 4 bytes  | YYYY-MM-DD          | 1000-01-01          | 9999-12-31          | 0000-00-00          |
| TIME         | 3 bytes  | HH:MM:SS            | -838:59:59          | 838:59:59           | 00:00:00            |
| YEAR         | 1 bytes  | YYYY                | 1901                | 2155                | 0000                |

Mysql保存日期格式使用 YYYY-MM-DD HH:MM:SS的ISO 8601标准,

创建字段

```sql
ALTER TABLE class ADD create_at datetime default null;
```

对于时间有一些数据格式化表示，通常有一些格式化参数，常用的格式化参数如下：更多的查询官方文档

| 参数 | 描述                               |
| ---- | ---------------------------------- |
| %Y   | 年，4 位                           |
| %y   | 年，2位                            |
| %M   | 月名                               |
| %m   | 月，数值(00-12)                    |
| %H   | 小时 (00-23)                       |
| %h   | 小时 (01-12)                       |
| %i   | 分钟，数值(00-59)                  |
| %s   | 秒(00-59)                          |
| %r   | 时间，12-小时（hh:mm:ss AM 或 PM） |

使用示例

```mysql
select cname,DATE_FORMAT(create_at,'%Y年%m月%d %H时%i分%s秒') as create_at from class;
select cname,TIME_FORMAT(create_at,'%r') as create_at from class;
```

**添加数据时自动更新时间**

```mysql
alter table class add updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP;
```

只要该条记录有任何一个字段被修改，则自动更新update_at字段的值

**常用函数**

| 函数                    | 说明                                                 |
| ----------------------- | ---------------------------------------------------- |
| HOUR                    | 小时                                                 |
| MINUTE                  | 分                                                   |
| SECOND                  | 秒                                                   |
| YEAR                    | 年                                                   |
| MONTH                   | 月                                                   |
| DAY                     | 日                                                   |
| TIME                    | 获取时间                                             |
| WEEK                    | 周                                                   |
| QUARTER                 | 季                                                   |
| CURRENT_DATE（CURDATE） | 当前日期                                             |
| CURRENT_TIME            | 当前时间                                             |
| NOW                     | 当前时间                                             |
| DAYOFYEAR               | 一年中的日数                                         |
| DAYOFMONTH              | 月份中日数                                           |
| DAYOFWEEK               | 星期天（1）到星期六（7）                             |
| WEEKDAY                 | 星期一（0）到星期天（6）                             |
| TO_DAYS                 | 从元年到现在的天数（忽略时间部分）                   |
| FROM_DAYS               | 根据天数得到日期（忽略时间部分）                     |
| TIME_TO_SEC             | 时间转为秒数（忽略日期部分）                         |
| SEC_TO_TIME             | 根据秒数转为时间（忽略日期部分）                     |
| UNIX_TIMESTAMP          | 根据日期返回秒数（包括日期与时间）                   |
| FROM_UNIXTIME           | 根据秒数返回日期与时间（包括日期与时间）             |
| DATEDIFF                | 两个日期相差的天数（忽略时间部分）                   |
| TIMEDIFF                | 计算两个时间的间隔（忽略日期部分）                   |
| TIMESTAMPDIFF           | 根据指定单位计算两个日期时间的间隔（包括日期与时间） |
| LAST_DAY                | 该月的最后一天                                       |

使用示例

```mysql
-- 使用函数分割时间
select cname,YEAR(create_at),MONTH(create_at),DAY(create_at),HOUR(create_at),MINUTE(create_at),SECOND(create_at) from class;
-- 当前时间
SELECT now(),CURDATE(),CURRENT_DATE(),CURRENT_TIME(),NOW();
-- 时间计算
SELECT DAYOFYEAR(now()),DAYOFMONTH(now()),DAYOFWEEK(now()),WEEKDAY(now());
```

**时间计算 常用函数 **

| 函数      | 说明                                                         |
| --------- | ------------------------------------------------------------ |
| ADDTIME   | 添加时间（负数为减少），只对时间有效                         |
| TIMESTAMP | 添加时间（负数为减少），只对时间有效                         |
| DATE_ADD  | 根据单位添加时间，支持单位有YEAR/MONTH/DAY/HOUR/MINUTE/SECOND/HOUR_MINUTE（负数时等于DATE_SUB) |
| DATE_SUB  | DATE_ADD的反函数                                             |
| LAST_DAY  | 指定月最后一天日期                                           |

使用示例

```mysql
-- 获取七小时之前的时间
select ADDTIME(now(),'-7:00:00')
-- 获取七天之后的时间，interval表示间隔
SELECT DATE_ADD(now(),INTERVAL 7 DAY);
-- 获取本月第一天的日期
SELECT DATE_SUB(now(),INTERVAL DAYOFMONTH(now())-1 DAY);
```

本文参考链接[后盾人MySQL教程]([http://houdunren.gitee.io/note/mysql/2%20%E5%9F%BA%E6%9C%AC%E6%93%8D%E4%BD%9C.html](http://houdunren.gitee.io/note/mysql/2 基本操作.html))

