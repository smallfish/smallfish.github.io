---
date: 2009-11-16
title: MySQL Query Profile 简单使用

tags:
- MySQL

---

MySQL Query Profile MySQL 5.0.37 以上开始支持 MySQL Query Profiler, 可以查询到此 SQL 会执行多少时间, 并看出 CPU/Memory 使用量, 执行过程中 System lock, Table lock 花多少时间等等.

详细可以参见官方文档：[http://dev.mysql.com/tech-resources/articles/using-new-query-profiler.html](http://dev.mysql.com/tech-resources/articles/using-new-query-profiler.html)

启动

{{< highlight python >}}
mysql> set profiling=1;
Query OK, 0 rows affected (0.00 sec)
{{< /highlight >}}

测试查询

{{< highlight python >}}
mysql> select count(*) from client where broker_id=2;
+----------+
| count(*) |
+----------+
|      200 |
+----------+
1 row in set (0.00 sec)
{{< /highlight >}}

查看profiles

{{< highlight python >}}
mysql> show profiles;
+----------+------------+-----------------------------------------------+
| Query_ID | Duration   | Query                                         |
+----------+------------+-----------------------------------------------+
|        0 | 0.00007300 | set profiling=1                               |
|        1 | 0.00044700 | select count(*) from client where broker_id=2 |
+----------+------------+-----------------------------------------------+
2 rows in set (0.00 sec)
{{< /highlight >}}

查看单条profile

{{< highlight python >}}
mysql> show profile for query 1;
+--------------------+------------+
| Status             | Duration   |
+--------------------+------------+
| (initialization)   | 0.00006300 |
| Opening tables     | 0.00001400 |
| System lock        | 0.00000600 |
| Table lock         | 0.00001000 |
| init               | 0.00002200 |
| optimizing         | 0.00001100 |
| statistics         | 0.00009300 |
| preparing          | 0.00001700 |
| executing          | 0.00000700 |
| Sending data       | 0.00016800 |
| end                | 0.00000700 |
| query end          | 0.00000500 |
| freeing items      | 0.00001200 |
| closing tables     | 0.00000800 |
| logging slow query | 0.00000400 |
+--------------------+------------+
15 rows in set (0.00 sec)

mysql> alter table t engine=myisam;
Query OK, 112050 rows affected (0.64 sec)
Records: 112050  Duplicates: 0  Warnings: 0

mysql> show profiles;
+----------+------------+-----------------------------------------------+
| Query_ID | Duration   | Query                                         |
+----------+------------+-----------------------------------------------+
|        0 | 0.00007300 | set profiling=1                               |
|        1 | 0.00044700 | select count(*) from client where broker_id=2 |
|        2 | 0.00003400 | set profiling=0                               |
|        3 | 0.00007400 | set profiling=1                               |
|        4 | 0.63789700 | alter table t engine=myisam                   |
|        5 | 0.00004000 | set profiling=0                               |
+----------+------------+-----------------------------------------------+
6 rows in set (0.00 sec)

mysql> show profile for query 4;
+----------------------+------------+
| Status               | Duration   |
+----------------------+------------+
| (initialization)     | 0.00002900 |
| checking permissions | 0.00000800 |
| init                 | 0.00004000 |
| Opening table        | 0.00009400 |
| System lock          | 0.00000500 |
| Table lock           | 0.00000700 |
| setup                | 0.00004200 |
| creating table       | 0.00195800 |
| After create         | 0.00010900 |
| copy to tmp table    | 0.52264500 |
| rename result table  | 0.11289400 |
| end                  | 0.00004600 |
| query end            | 0.00000700 |
| freeing items        | 0.00001300 |
+----------------------+------------+
14 rows in set (0.00 sec)
{{< /highlight >}}

查看cpu资源等信息

{{< highlight python >}}
mysql> show profile cpu for query 4;
+----------------------+------------+------------+------------+
| Status               | Duration   | CPU_user   | CPU_system |
+----------------------+------------+------------+------------+
| (initialization)     | 0.00002900 | 0.00000000 | 0.00000000 |
| checking permissions | 0.00000800 | 0.00000000 | 0.00000000 |
| init                 | 0.00004000 | 0.00000000 | 0.00000000 |
| Opening table        | 0.00009400 | 0.00100000 | 0.00000000 |
| System lock          | 0.00000500 | 0.00000000 | 0.00000000 |
| Table lock           | 0.00000700 | 0.00000000 | 0.00000000 |
| setup                | 0.00004200 | 0.00000000 | 0.00000000 |
| creating table       | 0.00195800 | 0.00000000 | 0.00100000 |
| After create         | 0.00010900 | 0.00000000 | 0.00000000 |
| copy to tmp table    | 0.52264500 | 0.55591600 | 0.04199300 |
| rename result table  | 0.11289400 | 0.00199900 | 0.00000000 |
| end                  | 0.00004600 | 0.00000000 | 0.00000000 |
| query end            | 0.00000700 | 0.00000000 | 0.00000000 |
| freeing items        | 0.00001300 | 0.00000000 | 0.00000000 |
+----------------------+------------+------------+------------+
14 rows in set (0.00 sec)
{{< /highlight >}}

其他属性列表

* ALL - displays all information
* BLOCK IO - displays counts for block input and output operations
* CONTEXT SWITCHES - displays counts for voluntary and involuntary context switches
* IPC - displays counts for messages sent and received
* MEMORY - is not currently implemented
* PAGE FAULTS - displays counts for major and minor page faults
* SOURCE - displays the names of functions from the source code, together with the name and line number of the file in which the function occurs
* SWAPS - displays swap counts

设定profiling保存size

{{< highlight python >}}
mysql> show variables where variable_name='profiling_history_size'; # 默认15条
{{< /highlight >}}

关闭

{{< highlight python >}}
mysql> set profiling=0;
{{< /highlight >}}


