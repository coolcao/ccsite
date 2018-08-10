---
title: MySQL事务隔离级别学习
date: 2018-08-10 22:40:27
tags: [MySQL, 事务隔离]
categories:
- 技术博客
- 笔记
---

# MySQL事务隔离级别学习笔记
## 隔离级别
1. READ UNCOMMITTED 未提交读
    在READ UNCOMMITTED级别，事务中的修改，即使没有提交，对其他事务也都是可见的。事务可以读取未提交的数据，这也被称作脏读。这个级别会导致很多问题，从性能上来说，READ UNCOMMITTED不会比其他的级别好太多，但确缺乏其他级别的很多好处，除非真的有必要的理由，在实际应用中一般很少使用。
2. READ COMMITED 提交读
    大多数数据库系统的默认隔离级别是 READ COMMITED（但mysql不是）。READ COMMITED满足前面提到的隔离性简单定义：一个事务开始时，只能看见已经提交的事务所做的修改。换句话说，一个事务从开始直到提交之前，所做的任何修改对其他事务都是不可见的。这个级别有时候也叫做不可重复读，因为两次执行同样的查询，可能会得到不一样的结果。
3. REPEATABLE READ 可重复写
    MySQL的默认事务隔离级别。
    REPEATABLE READ解决了脏读的问题。该级别保证了在同一个事务中多次读取同样的记录的结果是一致的。但是理论上，可重复读隔离级别还是无法解决另一个幻读的问题。所谓幻读，指的是当某个事务在读取某个范围内的记录时，另外一个事务又在该范围内插入了新的记录，当之前的事务再次读取该范围的记录时，会产生幻行。InnoDB存储引擎通过多版本并发控制（MVCC）解决了幻读的问题。
4. SERIALIZABLE 可串行化
    SERIALIZABLE 是最高的隔离级别。它通过强事务串行执行，避免了前面说的幻读的问题。简单来说，SERIALIZABLE会在曲度的每一行数据都加上锁，所以可能导致大量的超时和锁争用的问题。实际应用中也很少用到这个隔离级别，只有在非常需要确保数据一致性而且可以接受没有并发的情况下，才考虑使用该级别。

<!-- more -->

## 实验
上面是摘自《高性能MySQL》一书，光这么看，可能也理解不到多少东西，不如直接做实验亲身体验一下，加深理解。

我们先简单创建一个表 users，结构如下：

```
+------------+------------------+------+-----+-------------------+----------------+
| Field      | Type             | Null | Key | Default           | Extra          |
+------------+------------------+------+-----+-------------------+----------------+
| id         | int(11) unsigned | NO   | PRI | <null>            | auto_increment |
| name       | varchar(50)      | NO   |     |                   |                |
| created_at | datetime         | NO   |     | CURRENT_TIMESTAMP |                |
| updated_at | datetime         | NO   |     | CURRENT_TIMESTAMP |                |
| deleted_at | datetime         | YES  |     | <null>            |                |
+------------+------------------+------+-----+-------------------+----------------+
```

实现插入三条数据：
```
+----+-------------+---------------------+---------------------+------------+
| id | name        | created_at          | updated_at          | deleted_at |
+----+-------------+---------------------+---------------------+------------+
| 1  | coolcao2018 | 2018-05-28 12:48:43 | 2018-05-28 12:48:43 | <null>     |
| 2  | tom         | 2018-05-28 13:24:28 | 2018-05-28 13:24:28 | <null>     |
| 3  | lili        | 2018-05-31 08:54:28 | 2018-05-31 08:54:28 | <null>     |
+----+-------------+---------------------+---------------------+------------+
```

然后打开两个终端，A和B分别表示两个用户同时在操作。

### READ UNCOMMITTED
对于用户A，操作：
```
mysql root@localhost:test> set session transaction isolation level read uncommitted;
mysql root@localhost:test> start transaction;
mysql root@localhost:test> select * from users;
+----+-------------+---------------------+---------------------+------------+
| id | name        | created_at          | updated_at          | deleted_at |
+----+-------------+---------------------+---------------------+------------+
| 1  | coolcao2018 | 2018-05-28 12:48:43 | 2018-05-28 12:48:43 | <null>     |
| 2  | tom         | 2018-05-28 13:24:28 | 2018-05-28 13:24:28 | <null>     |
| 3  | lili        | 2018-05-31 08:54:28 | 2018-05-31 08:54:28 | <null>     |
+----+-------------+---------------------+---------------------+------------+
```
然后，用户B操作，
```
mysql root@localhost:test> set session transaction isolation level read uncommitted;
mysql root@localhost:test> start transaction;
mysql root@localhost:test> update users set name='coolcao' where id=1;
```
此时，用户B并未提交事务，用户A进行查询操作看看：
```
mysql root@localhost:test> select * from users;
+----+---------+---------------------+---------------------+------------+
| id | name    | created_at          | updated_at          | deleted_at |
+----+---------+---------------------+---------------------+------------+
| 1  | coolcao | 2018-05-28 12:48:43 | 2018-05-28 12:48:43 | <null>     |
| 2  | tom     | 2018-05-28 13:24:28 | 2018-05-28 13:24:28 | <null>     |
| 3  | lili    | 2018-05-31 08:54:28 | 2018-05-31 08:54:28 | <null>     |
+----+---------+---------------------+---------------------+------------+
```
从整个过程来看，用户B还并未提交事务，但是A却已经能够直接读到B的更新。

> 从上面实验结果来看，不难理解上面对于 READ UNCOMMITTED级别的描述：*在READ UNCOMMITTED级别，事务中的修改，即使没有提交，对其他事务也都是可见的。*

### READ COMMITTED
同时将A，B两个终端的事务级别设置为 read committed:
```
 // 在A，B两个终端都执行
 set session transaction isolation level read committed;
```
对于A，我们开启一个事务，然后更新一下数据，但并不提交事务：
```
mysql root@localhost:test>  set session transaction isolation level read committed;
mysql root@localhost:test> start transaction;
mysql root@localhost:test> select * from users;
+----+---------+---------------------+---------------------+------------+
| id | name    | created_at          | updated_at          | deleted_at |
+----+---------+---------------------+---------------------+------------+
| 1  | coolcao | 2018-05-28 12:48:43 | 2018-05-28 12:48:43 | <null>     |
| 2  | tom     | 2018-05-28 13:24:28 | 2018-05-28 13:24:28 | <null>     |
| 3  | lili    | 2018-05-31 08:54:28 | 2018-05-31 08:54:28 | <null>     |
+----+---------+---------------------+---------------------+------------+
mysql root@localhost:test> update users set name='coolcao222' where id=1;
mysql root@localhost:test> select * from users;
+----+------------+---------------------+---------------------+------------+
| id | name       | created_at          | updated_at          | deleted_at |
+----+------------+---------------------+---------------------+------------+
| 1  | coolcao222 | 2018-05-28 12:48:43 | 2018-05-28 12:48:43 | <null>     |
| 2  | tom        | 2018-05-28 13:24:28 | 2018-05-28 13:24:28 | <null>     |
| 3  | lili       | 2018-05-31 08:54:28 | 2018-05-31 08:54:28 | <null>     |
+----+------------+---------------------+---------------------+------------+
```

然后，在B终端，开启另外一个事务，进行数据查询：
```
mysql root@localhost:test> set session transaction isolation level read committed;
mysql root@localhost:test> start transaction;
mysql root@localhost:test> select * from users;
+----+---------+---------------------+---------------------+------------+
| id | name    | created_at          | updated_at          | deleted_at |
+----+---------+---------------------+---------------------+------------+
| 1  | coolcao | 2018-05-28 12:48:43 | 2018-05-28 12:48:43 | <null>     |
| 2  | tom     | 2018-05-28 13:24:28 | 2018-05-28 13:24:28 | <null>     |
| 3  | lili    | 2018-05-31 08:54:28 | 2018-05-31 08:54:28 | <null>     |
+----+---------+---------------------+---------------------+------------+
```

然后，将A事务提交：
```
commit;
```

这时，再在B查询 ：
```
mysql root@localhost:test> select * from users;
+----+------------+---------------------+---------------------+------------+
| id | name       | created_at          | updated_at          | deleted_at |
+----+------------+---------------------+---------------------+------------+
| 1  | coolcao222 | 2018-05-28 12:48:43 | 2018-05-28 12:48:43 | <null>     |
| 2  | tom        | 2018-05-28 13:24:28 | 2018-05-28 13:24:28 | <null>     |
| 3  | lili       | 2018-05-31 08:54:28 | 2018-05-31 08:54:28 | <null>     |
+----+------------+---------------------+---------------------+------------+
```

从结果来看，也不难理解 read committed级别，对于一个事务，只能读取到当前事务的数据和其他已经提交的事务的数据，对于其他未提交事务的数据，读不到。
而且，从上面的实验结果中，我们也看到了，会话B在会话A提交事务前后查询的结果并不一致，这也就是上面所说的，不可重复读。

### REPEATABLE READ 可重复读

我们将会话A设置为REPEATABLE READ :
```
mysql root@localhost:test> set session transaction isolation level repeatable read;
mysql root@localhost:test> start transaction;
mysql root@localhost:test> select * from users;
+----+---------+---------------------+---------------------+------------+
| id | name    | created_at          | updated_at          | deleted_at |
+----+---------+---------------------+---------------------+------------+
| 1  | coolcao | 2018-08-10 15:21:02 | 2018-08-10 15:21:02 | <null>     |
| 2  | tom     | 2018-08-10 15:21:07 | 2018-08-10 15:21:07 | <null>     |
| 3  | lili    | 2018-08-10 15:21:11 | 2018-08-10 15:21:11 | <null>     |
+----+---------+---------------------+---------------------+------------+
```

此时，我们在B终端插入一条数据：
```
mysql root@localhost:test> insert into users (id,name) values (4,'coco');
mysql root@localhost:test> commit;
mysql root@localhost:test> select * from users;
+----+---------+---------------------+---------------------+------------+
| id | name    | created_at          | updated_at          | deleted_at |
+----+---------+---------------------+---------------------+------------+
| 1  | coolcao | 2018-08-10 15:21:02 | 2018-08-10 15:21:02 | <null>     |
| 2  | tom     | 2018-08-10 15:21:07 | 2018-08-10 15:21:07 | <null>     |
| 3  | lili    | 2018-08-10 15:21:11 | 2018-08-10 15:21:11 | <null>     |
| 4  | coco    | 2018-08-10 15:23:50 | 2018-08-10 15:23:50 | <null>     |
+----+---------+---------------------+---------------------+------------+
```

在终端B中，插入一条记录，并提交，这时id=4的用户已经被插入到数据库。

此时，再回到终端A，查询：
```
mysql root@localhost:test> select * from users;
+----+---------+---------------------+---------------------+------------+
| id | name    | created_at          | updated_at          | deleted_at |
+----+---------+---------------------+---------------------+------------+
| 1  | coolcao | 2018-08-10 15:21:02 | 2018-08-10 15:21:02 | <null>     |
| 2  | tom     | 2018-08-10 15:21:07 | 2018-08-10 15:21:07 | <null>     |
| 3  | lili    | 2018-08-10 15:21:11 | 2018-08-10 15:21:11 | <null>     |
+----+---------+---------------------+---------------------+------------+
```
哎，查询的结果中，没有B刚插入的id=4的用户，这也就是说该级别的事务隔离，保证了在同一个事务中多次读取同样的记录的结果是一致的。这时，我们在A中插入一条记录：
```
mysql root@localhost:test> insert into users (id,name) values (4,'coco');
(1062, u"Duplicate entry '4' for key 'PRIMARY'")
```
哎，这个时候，数据库报错了，提示主键重复。明明我在这个事务中，查询的数据只有1,2,3，为什么插入4的时候提示主键冲突呢？是发生幻觉了么？是的，发生“幻读”了。由于REPEATABLE READ级别的隔离，在一个事务中，多次读取同样记录的结果是一致的，在这多次读取之间，被别的事务插入了新的数据，这时前事务再插入数据，必然会导致错误。

### SERIALIZABLE 可串行化

我们将A，B同时设置为SERIALIZABLE, 然后在A开启是个事务，做一个简单查询：

```
mysql root@localhost:test> set session transaction isolation level serializable;
mysql root@localhost:test> start transaction;
mysql root@localhost:test> select * from users where id<10;
+----+---------+---------------------+---------------------+------------+
| id | name    | created_at          | updated_at          | deleted_at |
+----+---------+---------------------+---------------------+------------+
| 1  | coolcao | 2018-08-10 15:21:02 | 2018-08-10 15:21:02 | <null>     |
| 2  | tom     | 2018-08-10 15:21:07 | 2018-08-10 15:21:07 | <null>     |
| 3  | lili    | 2018-08-10 15:21:11 | 2018-08-10 15:21:11 | <null>     |
| 4  | coco    | 2018-08-10 15:23:50 | 2018-08-10 15:23:50 | <null>     |
| 5  | juli    | 2018-08-10 15:31:32 | 2018-08-10 15:31:32 | <null>     |
+----+---------+---------------------+---------------------+------------+
```

此时，A事务并未提交，然后在B再开启一个事务，进行插入操作：

```
mysql root@localhost:test> set session transaction isolation level serializable;
mysql root@localhost:test> start transaction;
mysql root@localhost:test> insert into users (id,name) values (6,'kate');
(1205, u'Lock wait timeout exceeded; try restarting transaction')
```
你会发现，哎我去，B事务被挂住了，然后过了一段时间，提示了错误 `(1205, u'Lock wait timeout exceeded; try restarting transaction')`，说等待锁超时。
是的，在串行化级别，会在读取的每一行数据都加上锁，也就是说，上面A事务在读取时，已经加了锁，此时B事务在插入操作时，得等待锁的放开，时间一长，A锁未放开，B就报错了。
从实验中可以看出，可串行化级别，由于要保证避免幻读而加了锁导致效率以及可能会触发的等待锁超时等错误，实际应用中，该级别的事务隔离也很少使用。

对照着实验结果，来理解上面四个隔离级别，就容易理解了。


