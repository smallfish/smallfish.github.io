---
date: 2010-02-05
title: MySQL/PostgreSQL小命令对比

tags:
- PostgreSQL

---

对比下一些两个数据库常用的操作。分别使用自带的client程序。

MySQL命令行：

{{< highlight python >}}
mysql -u 用户名 -h 主机地址 -P 端口号 数据库名 -p
{{< /highlight >}}

PostgreSQL命令行：

{{< highlight python >}}
psql -U 用户名 -h 主机地址 -p 端口号 数据库名
{{< /highlight >}}

操作对比：

{{< highlight python >}}
mysql                      psql

SHOW DATABASES;           \l
USE db-name;              \c db-name
SHOW TABLES;              \d
SHOW USERS;               \du
SHOW COLUMNS;             \d table-name
SHOW PROCESSLIST;         SELECT * FROM pg_stat_activity;
SELECT now()\G            \x 可以打开和关闭类似\G功能
SOURCE /path.sql          \i /path.sql
LOAD DATA INFILE ...      \copy ...
\h                        \?
{{< /highlight >}}


