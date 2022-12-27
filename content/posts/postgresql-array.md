---
date: 2013-11-28
title: PostgreSQL ARRAY 数据类型
---

刚好手头有一个需求，是涉及到数组类型的，懒的插入多条数据库记录，想起了ARRAY数据类型。

官方文档参考：

* [8.14. Arrays](http://www.postgresql.org/docs/9.3/static/arrays.html)
* [9.17. Array Functions and Operators](http://www.postgresql.org/docs/9.3/static/functions-array.html)

这里简单的介绍下和一些示例，完整的还是推荐参考官方吧。测试版本为：PostgreSQL 9.3

比如声明integer/varchar数组：

{{< highlight sql >}}
x integer[5]
y varchar[]
z varchar ARRAY
{{< /highlight >}}

后两个声明等价，懒人的话就别指明ARRAY长度了，建一个表来玩玩吧。

{{< highlight sql >}}
CREATE TABLE test1 (
  x integer[5],
  y varchar[],
  z varchar ARRAY
);

smallfish=# \d test1
       Table "public.test1"
 Column |        Type         | Modifiers
--------+---------------------+-----------
 x      | integer[]           |
 y      | character varying[] |
 z      | character varying[] |
 
没骗你们吧，后两者完全一样。
{{< /highlight >}}

写入ARRAY字段，有一下两种方式：

* 显示声明，比如：ARRAY[11, 22]
* 花括号，比如：'{aa, bb}'

插入一条记录：

{{< highlight sql >}}
INSERT INTO test1 VALUES (ARRAY[11, 22], '{aa, bb}', ARRAY['cc', 'dd']);
{{< /highlight >}}

查询一下，看看表里出来是啥样子：

{{< highlight sql >}}
smallfish=# SELECT * FROM test1;
    x    |    y    |    z
---------+---------+---------
 {11,22} | {aa,bb} | {cc,dd}
{{< /highlight >}}

是不是看上去还挺帅的样子，这里的字符串没有特殊符号，比如空格斜杠之类的，可以双引号或用E来包一下。

ARRAY索引是从1开始的，这点稍微区别下一般语言的数组，还支持类似切片的查询哦。

{{< highlight sql >}}
smallfish=# SELECT x[1], y, z FROM test1 WHERE x[2]>10;
 x  |    y    |    z
----+---------+---------
 11 | {aa,bb} | {cc,dd}
 
smallfish=# SELECT x[1:2] FROM test1;
    x
---------
 {11,22}
{{< /highlight >}}

说了读，写，还有更新的例子：

{{< highlight sql >}}
smallfish=# UPDATE test1 SET x[1] = 100;
smallfish=# SELECT * FROM test1;
    x     |    y    |    z
----------+---------+---------
 {100,22} | {aa,bb} | {cc,dd}
{{< /highlight >}}

给数组添加新元素：

{{< highlight sql >}}
smallfish=# UPDATE test1 SET x[3] = 999;
smallfish=# SELECT * FROM test1;
      x       |    y    |    z
--------------+---------+---------
 {100,22,999} | {aa,bb} | {cc,dd}
{{< /highlight >}}

按照索引删除的功能似乎要写函数来解决了，官方有一个array_remove的函数，不过这个是按照value来移除的，如：

{{< highlight sql >}}
smallfish=# UPDATE test1 SET x = array_remove(x, 22);
smallfish=# SELECT x FROM test1;
     x
-----------
 {100,999}
{{< /highlight >}}

这年头能用上Pg真心不容易，用的还是9.x版本。谢天谢地~





