---
date: 2011-02-19
title: PostgreSQL Key-Value 数据类型 hstore 使用教程
---

现在满地都是KV数据库的文字，PostgreSQL 也有类似的结构。不过是通过其强大的扩展方式实现的。

官网文档请参考：[http://www.postgresql.org/docs/current/static/hstore.html](http://www.postgresql.org/docs/current/static/hstore.html)

本文测试环境在 Mac OS 下，Pg采用源码编译。

编译 hstore 扩展

{{< highlight python >}}
mac:~ smallfish$ cd Downloads/postgresql-9.0.1/contrib/hstore/
mac:hstore smallfish$ make
... 一堆编译信息
mac:hstore smallfish$ sudo make install
{{< /highlight >}}

导入到数据库中，注意必须以 postgres 用户，如果需要装入到指定数据库，请指明。这里采用默认数据库。

{{< highlight python >}}
mac:hstore smallfish$ ls /opt/postgresql/share/contrib/
hstore.sql		uninstall_hstore.sql
mac:hstore smallfish$ psql -U postgres -f /opt/postgresql/share/contrib/hstore.sql
... 一堆导入命令
{{< /highlight >}}

进入数据库，建一个测试表

{{< highlight python >}}
postgres=# CREATE TABLE testhstore (id SERIAL, value hstore);
NOTICE:  CREATE TABLE will create implicit sequence "testhstore_id_seq" for serial column "testhstore.id"
CREATE TABLE
{{< /highlight >}}

查看下表结构

{{< highlight python >}}
postgres=# \d
                List of relations
 Schema |       Name        |   Type   |  Owner
--------+-------------------+----------+----------
 public | testhstore        | table    | postgres
 public | testhstore_id_seq | sequence | postgres
(2 rows)

postgres=# \d testhstore;
                         Table "public.testhstore"
 Column |  Type   |                        Modifiers
--------+---------+---------------------------------------------------------
 id     | integer | not null default nextval('testhstore_id_seq'::regclass)
 value  | hstore  |
{{< /highlight >}}

尝试下简单 hstore类型

{{< highlight python >}}
postgres=# select 'a=>1, b=>2'::hstore;
       hstore
--------------------
 "a"=>"1", "b"=>"2"
(1 row)

postgres=# select 'a=>1, b=>a'::hstore;
       hstore
--------------------
 "a"=>"1", "b"=>"a"
(1 row)
{{< /highlight >}}

写几条测试数据先

{{< highlight python >}}
postgres=# INSERT INTO testhstore (value) VALUES ('name=>smallfish, age=>29'::hstore);
INSERT 0 1
postgres=# SELECT * FROM testhstore;
 id |              value
----+----------------------------------
  1 | "age"=>"29", "name"=>"smallfish"
(1 row)

postgres=# INSERT INTO testhstore (value) VALUES ('name=>nnfish, age=>20'::hstore);
INSERT 0 1
postgres=# INSERT INTO testhstore (value) VALUES ('name=>aaa, age=>30, addr=>China'::hstore);
INSERT 0 1
{{< /highlight >}}

查看下所有数据

{{< highlight python >}}
postgres=# SELECT * FROM testhstore;
 id |                    value
----+---------------------------------------------
  1 | "age"=>"29", "name"=>"smallfish"
  2 | "age"=>"20", "name"=>"nnfish"
  3 | "age"=>"30", "addr"=>"China", "name"=>"aaa"
(3 rows)
{{< /highlight >}}

查询列里面的指定 key

{{< highlight python >}}
postgres=# SELECT id, value->'name' AS name FROM testhstore;
 id |   name
----+-----------
  1 | smallfish
  2 | nnfish
  3 | aaa
(3 rows)

postgres=# SELECT id, value->'name', value->'age' AS age FROM testhstore;
 id | ?column?  | age
----+-----------+-----
  1 | smallfish | 29
  2 | nnfish    | 20
  3 | aaa       | 30
(3 rows)
{{< /highlight >}}

修改列某 key 值

{{< highlight python >}}
postgres=# UPDATE testhstore SET value=value||('addr=>Shanghai') WHERE id = 2;
UPDATE 1

postgres=# SELECT * FROM testhstore;
 id |                       value
----+---------------------------------------------------
  1 | "age"=>"29", "name"=>"smallfish"
  3 | "age"=>"30", "addr"=>"China", "name"=>"aaa"
  2 | "age"=>"20", "addr"=>"Shanghai", "name"=>"nnfish"
(3 rows)
{{< /highlight >}}

删除列里某 key

{{< highlight python >}}
postgres=# UPDATE testhstore SET value=delete(value, 'addr') WHERE id = 3;
UPDATE 1

postgres=# SELECT * FROM testhstore;
 id |                       value
----+---------------------------------------------------
  1 | "age"=>"29", "name"=>"smallfish"
  2 | "age"=>"20", "addr"=>"Shanghai", "name"=>"nnfish"
  3 | "age"=>"30", "name"=>"aaa"
(3 rows)
{{< /highlight >}}

按条件查询列里某 key ，注意要数据类型，CAST 强转。默认都是字符串的值。

{{< highlight python >}}
postgres=# SELECT * FROM testhstore WHERE (value->'age')::INT > 20;
 id |              value
----+----------------------------------
  1 | "age"=>"29", "name"=>"smallfish"
  3 | "age"=>"30", "name"=>"aaa"
(2 rows)

postgres=# SELECT * FROM testhstore WHERE value->'name' = 'smallfish';
 id |              value
----+----------------------------------
  1 | "age"=>"29", "name"=>"smallfish"
(1 row)
{{< /highlight >}}

&nbsp;

