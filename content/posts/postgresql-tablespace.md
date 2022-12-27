---
date: 2009-12-22
title: PostgreSQL tablespace 表空间

tags:
- PostgreSQL

---

pgsql允许管理员在文件系统里定义表空间存储位置，这样创建数据库对象时候就可以引用这个表空间了。好处就不用多说了，可以把数据库对象存储到不同的分区上，比如更好的存储之类。默认initdb之后会有两个表空间pg_global和pg_default。

查看pgsql当前表空间有哪些可以试试下面：

{{< highlight python >}}

postgres=> SELECT spcname FROM pg_tablespace;
  spcname
------------
 pg_default
 pg_global
(2 rows)

{{< /highlight >}}

或：

{{< highlight python >}}

postgres=> \db
    Name    |  Owner   | Location
------------+----------+----------
 pg_default | postgres |
 pg_global  | postgres |

{{< /highlight >}}

建立表空间需要注意的主要的是权限问题，而且要在新的空目录上建立，权限属于数据库管理员比如默认postgres。

1. 建立目录

{{< highlight python >}}

$ mkdir /home/smallfish/pgdata
$ sudo chown -R postgres:postgres /home/smallfish/pgdata

{{< /highlight >}}

2. 进入psql

{{< highlight python >}}

$ psql -U postgres -h 192.168.0.122

{{< /highlight >}}

如果权限没设置好下面语句会报错

{{< highlight python >}}

postgres=> CREATE TABLESPACE space1 LOCATION '/home/smallfish/pgdata';

{{< /highlight >}}

建测试表

{{< highlight python >}}

postgres=> CREATE TABLE foo(i int) TABLESPACE space1;

{{< /highlight >}}

可以查看表空间目录下多了文件

{{< highlight python >}}

postgres=> \! ls /home/smallfish/pgdata

{{< /highlight >}}

删除表空间，需要注意的是先要删除所有该表空间里的对象

{{< highlight python >}}

postgres=> DROP TABLESPACE space1;

{{< /highlight >}}

ok，到这里已经建立好表空间了。当然每次建表都指定TABLESPACE也有点麻烦，来点默认的把。

{{< highlight python >}}

postgres=> SET default_tablespace = space1;
postgres=> CREATE TABLE foo(i int);

{{< /highlight >}}


