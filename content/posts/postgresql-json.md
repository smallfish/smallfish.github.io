---
date: 2014-07-25
title: PostgreSQL JSON 数据类型
---

从PostgreSQL 9.3版本开始，JSON已经成为内置数据类型，“一等公民”啦。

还在羡慕什么文档数据库或者BSON么，赶紧玩玩吧。另外9.4版本，提供JSONB（Binary），提供更多JSON函数和索引支持。

刚好手头有一个需求，是涉及到数组类型的，懒的插入多条数据库记录，想起了ARRAY数据类型。

常用的读取操作符目前大概有三类：`->`、`->>`和`#>`。还是直接看SQL查询的例子吧。

先看`->`类：

{{< highlight sql >}}
postgres=# select '[1,2,3]'::json->2;
 ?column?
----------
 3
(1 row)

postgres=# select '{"a":1,"b":2}'::json->'b';
 ?column?
----------
 2
(1 row)
{{< /highlight >}}

再来`->>`例子：

{{< highlight sql >}}
postgres=# select '[1,2,3]'::json->>2;
 ?column?
----------
 3
(1 row)

postgres=# select '{"a":1,"b":2}'::json->>'b';
 ?column?
----------
 2
(1 row)
{{< /highlight >}}

有没有发现其实`->`和`->>`出来的结果肉眼看起来是一样的？区别在于后者是返回text。

上面两个操作符实现了读取，其实大部分时候我们的JSON不是这么简单，会内嵌各种数组和哈希，这么读下去会死人的吧。当然，有一种类似path的读取，看例子吧：

{{< highlight sql >}}
postgres=# select '{"a":[1,2,3],"b":[4,5,6]}'::json#>'{a,2}';
 ?column?
----------
 3
(1 row)

postgres=# select '{"a":[1,2,3],"b":[4,5,6]}'::json#>>'{a,2}';
 ?column?
----------
 3
(1 row)
{{< /highlight >}}

当然里面可以嵌套很多，比如`{a,2,b,3}`等等。下面再来点表的例子：

{{< highlight sql >}}
postgres=# create table testjson(id serial, data json);
postgres=# insert into testjson (data) values('{"a": 1, "b": 2}'::json);
postgres=# insert into testjson (data) values('{"a": 3, "b": 4, "c": 5}'::json);
postgres=# insert into testjson (data) values('{"a": 6, "c": 7}'::json);
{{< /highlight >}}

插入数据是不是很熟悉，基本和普通使用JSON一致，初窥下`select`结果：

{{< highlight sql >}}
postgres=# select * from testjson;
 id |           data
----+--------------------------
  1 | {"a": 1, "b": 2}
  2 | {"a": 3, "b": 4, "c": 5}
  3 | {"a": 6, "c": 7}
(3 rows)
{{< /highlight >}}

很眼熟，`where`条件可以这么用：

{{< highlight sql >}}
postgres=# select * from testjson where (data->>'a')::int>1;
 id |           data
----+--------------------------
  2 | {"a": 3, "b": 4, "c": 5}
  3 | {"a": 6, "c": 7}
(2 rows)
{{< /highlight >}}

注意这里是`->>`转换成text然后在`::int`进行比较。

不过这里有个坑，不知道怎么解决，比如：

{{< highlight sql >}}
postgres=# insert into testjson (data) values('{"a": "smallfish"}');
postgres=# select * from testjson;
 id |           data
----+--------------------------
  1 | {"a": 1, "b": 2}
  2 | {"a": 3, "b": 4, "c": 5}
  3 | {"a": 6, "c": 7}
  5 | {"a": "smallfish"}
(4 rows)
{{< /highlight >}}

这个时候上面的`::int>1`这样就报错了，因为最后一行`a`为字符串。

mongodb这个问题解决的挺好，回头搜搜在Pg里怎么搞。

话说，Pg里支持的JSON是不是和之前提到的[hstore](http://chenxiaoyu.org/2011/02/19/postgresql-key-value-hstore.html)类型有的一相似的地方？

两者优劣回头我再贴文吧。

官方文档：[http://www.postgresql.org/docs/9.3/static/functions-json.html](http://www.postgresql.org/docs/9.3/static/functions-json.html)





