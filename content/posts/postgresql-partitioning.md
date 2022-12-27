---
date: 2009-12-22
title: PostgreSQL Partitioning 表分区
---

测试版本：pg 8.3 (ubuntu)

在pg里表分区是通过表继承来实现的，一般都是建立一个主表，里面是空，然后每个分区都去继承它。

创建表分区步骤如下：

1. 创建主表

{{< highlight python >}}
CREATE TABLE users ( uid int not null primary key, name varchar(20));
{{< /highlight >}}

2. 创建分区表(必须继承上面的主表)

{{< highlight python >}}
CREATE TABLE users_0 ( check (uid >= 0 and uid< 100) ) INHERITS (users);
CREATE TABLE users_1 ( check (uid >= 100)) INHERITS (users);
{{< /highlight >}}

3. 在分区表上建立索引，其实这步可以省略的哦

{{< highlight python >}}
CREATE INDEX users_0_uidindex on users_0(uid);
CREATE INDEX users_1_uidindex on users_1(uid);
{{< /highlight >}}

4. 创建规则RULE

{{< highlight python >}}
CREATE RULE users_insert_0 AS
    ON INSERT TO users WHERE
        (uid >= 0 and uid < 100)
    DO INSTEAD
        INSERT INTO users_0 VALUES (NEW.uid,NEW.name);

CREATE RULE users_insert_1 AS
    ON INSERT TO users WHERE
        (uid >= 100)
    DO INSTEAD
        INSERT INTO users_1 VALUES (NEW.uid,NEW.name);
{{< /highlight >}}

下面就可以测试写入数据啦：

{{< highlight python >}}
postgres=# INSERT INTO users VALUES (100,'smallfish');
INSERT 0 0
postgres=# INSERT INTO users VALUES (20,'aaaaa');
INSERT 0 0
postgres=# select * from users;
uid  |   name
-----+-----------
20   | aaaaa
100  | smallfish
(2 笔资料列)
postgres=# select * from users_0;
uid  | name
-----+-------
20   | aaaaa
(1 笔资料列)

postgres=# select * from users_1;
uid  |   name
-----+-----------
100  | smallfish
(1 笔资料列)
{{< /highlight >}}

到这里表分区已经可以算完了，不过还有个地方需要修改下，先看count查询把。

{{< highlight python >}}
postgres=# EXPLAIN SELECT count(*) FROM users where uid<100;
QUERY PLAN
------------------------------------------------
Aggregate  (cost=62.75..62.76 rows=1 width=0)
    ->  Append  (cost=6.52..60.55 rows=879 width=0)
    ->  Bitmap Heap Scan on users  (cost=6.52..20.18 rows=293 width=0)
Recheck Cond: (uid < 100)
    ->  Bitmap Index Scan on users_pkey  (cost=0.00..6.45 rows=293 width=0)
Index Cond: (uid < 100)
    ->  Bitmap Heap Scan on users_0 users  (cost=6.52..20.18 rows=293 width=0)
Recheck Cond: (uid < 100)
    ->  Bitmap Index Scan on users_0_uidindex  (cost=0.00..6.45 rows=293 width=0)
Index Cond: (uid < 100)
    ->  Bitmap Heap Scan on users_1 users  (cost=6.52..20.18 rows=293 width=0)
Recheck Cond: (uid < 100)
    ->  Bitmap Index Scan on users_1_uidindex  (cost=0.00..6.45 rows=293 width=0)
Index Cond: (uid < 100)
(14 笔资料列)
{{< /highlight >}}

按照本来想法，uid小于100，理论上应该只是查询users_0表，通过EXPLAIN可以看到其他他扫描了所有分区的表。

{{< highlight python >}}
postgres=# SET constraint_exclusion = on;
SET

{{< /highlight >}}


{{< highlight python >}}

postgres=# EXPLAIN SELECT count(*) FROM users where uid<100;
QUERY PLAN
------------------------------------------------
Aggregate  (cost=41.83..41.84 rows=1 width=0)
    ->  Append  (cost=6.52..40.37 rows=586 width=0)
    ->  Bitmap Heap Scan on users  (cost=6.52..20.18 rows=293 width=0)
Recheck Cond: (uid < 100)
    ->  Bitmap Index Scan on users_pkey  (cost=0.00..6.45 rows=293 width=0)
Index Cond: (uid < 100)
    ->  Bitmap Heap Scan on users_0 users  (cost=6.52..20.18 rows=293 width=0)
Recheck Cond: (uid < 100)
    ->  Bitmap Index Scan on users_0_uidindex  (cost=0.00..6.45 rows=293 width=0)
Index Cond: (uid < 100)
(10 笔资料列)
{{< /highlight >}}

到这里整个过程都OK啦！

