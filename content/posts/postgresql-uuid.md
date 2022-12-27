---
date: 2010-02-26
title: PostgreSQL UUID 函数

tags:
- PostgreSQL

---

测试环境：PostgreSQL 8.4

默认PostgreSQL是木有UUID函数可使用，而不像MySQL提供uuid()函数，不过在contrib里有，只需要导入一下uuid-ossp.sql即可。（PS：注意权限问题，要Pg可读改文件。）

导入很简单，下面是win下面测试，其他平台类似该操作：

{{< highlight python >}}
D:\>psql -U postgres -h localhost -f D:\PostgreSQL\8.4\share\contrib\uuid-ossp.sql
Password for user postgres:
SET
CREATE FUNCTION
CREATE FUNCTION
CREATE FUNCTION
CREATE FUNCTION
CREATE FUNCTION
CREATE FUNCTION
CREATE FUNCTION
CREATE FUNCTION
CREATE FUNCTION
CREATE FUNCTION
{{< /highlight >}}

进入psql，执行：

{{< highlight python >}}
postgres=# select uuid_generate_v1();
           uuid_generate_v1
--------------------------------------
 86811bd4-22a5-11df-b00e-ebd863f5f8a7
(1 row)

postgres=# select uuid_generate_v4();
           uuid_generate_v4
--------------------------------------
 5edbfcbb-1df8-48fa-853f-7917e4e346db
(1 row)
{{< /highlight >}}

主要就是uuid_generate_v1和uuid_generate_v4，当然还有uuid_generate_v3和uuid_generate_v5。其他使用可以参见PostgreSQL官方文档 [http://www.postgresql.org/docs/8.3/static/uuid-ossp.html](http://www.postgresql.org/docs/8.3/static/uuid-ossp.html)。

