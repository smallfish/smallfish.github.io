---
date: 2010-06-02
title: PostgreSQL COPY 导入/导出数据

tags:
- PostgreSQL

---

COPY 命令可以快速的导入数据到 PostgreSQL 中，文件格式类似CVS之类。适合批量导入数据，比 \i 和恢复数据表快。

导出表数据到文件或 STDOUT ：

{{< highlight python >}}
COPY tablename [(column [, ...])]
   TO {'filename' | STDOUT}
   [[WITH]
      [BINARY]
      [OIDS]
      [DELIMITER [AS] 'delimiter']
      [NULL [AS] 'null string']
      [CSV [HEADER]
         [QUOTE [AS] 'quote']
         [ESCAPE [AS] 'escape']
         [FORCE NOT NULL column [, ...]]
{{< /highlight >}}

导入文件或者 STDIN 到表中：

{{< highlight python >}}
COPY tablename [(column [, ...])]
   FROM {'filename' | STDIN}
   [[WITH]
      [BINARY]
      [OIDS]
      [DELIMITER [AS] 'delimiter']
      [NULL [AS] 'null string']
      [CSV [HEADER]
         [QUOTE [AS] 'quote']
         [ESCAPE [AS] 'escape']
         [FORCE QUOTE column [, ...]]
{{< /highlight >}}

导出表 employee 到默认输出 STDOUT：

{{< highlight python >}}
psql> COPY employee TO STDOUT;
1       JG100011        Jason Gilmore         jason@example.com
2       RT435234        Robert Treat          rob@example.com
3       GS998909        Greg Sabino Mullane   greg@example.com
4       MW777983        Matt Wade             matt@example.com
{{< /highlight >}}

导出表 employee 到 sql 文件：

{{< highlight python >}}
psql> COPY employee TO '/home/smallfish/employee.sql';
{{< /highlight >}}

从文件导入数据：

{{< highlight python >}}
psql> COPY employeenew FROM '/home/smallfish/employee.sql';
psql> SELECT * FROM employeenew;
employeeid  | employeecode |     name            |       email
------------+--------------+---------------------+---------------
          1 | JG100011     | Jason Gilmore       | jason@example.com
          2 | RT435234     | Robert Treat        | rob@example.com
          3 | GS998909     | Greg Sabino Mullane | greg@example.com
          4 | MW777983     | Matt Wade           | matt@example.com
(4 rows)
{{< /highlight >}}

输出对象ID（OIDS）：

{{< highlight python >}}
psql> COPY employee TO STDOUT OIDS;
24627  1       GM100011        Jason Gilmore         jason@example.com
24628  2       RT435234        Robert Treat          rob@example.com
24629  3       GS998909        Greg Sabino Mullane   greg@example.com
24630  4       MW777983        Matt Wade             matt@example.com
{{< /highlight >}}

指定导出间隔符，默认是 \t ，这里为 | ：

{{< highlight python >}}
psql>COPY employee TO STDOUT DELIMITER '|';
1|GM100011|Jason Gilmore|jason@example.com
2|RT435234|Robert Treat|rob@example.com
3|GS998909|Greg Sabino Mullane|greg@example.com
4|MW777983|Matt Wade|matt@example.com
{{< /highlight >}}

导入文件数据，指定间隔符为 | ：

{{< highlight python >}}
psql> COPY employeenew FROM '/home/smallfish/employee.sql' DELIMITER |;
{{< /highlight >}}

导出指定字段的数据：

{{< highlight python >}}
psql> COPY employee (name,email) TO STDOUT;
Jason Gilmore         jason@example.com
Robert Treat          rob@example.com
Greg Sabino Mullane   greg@example.com
Matt Wade             matt@example.com
{{< /highlight >}}

为 NULL 字段设置默认值：

{{< highlight python >}}
psql> COPY employee TO STDOUT NULL 'no email';
Jason Gilmore         no email
Robert Treat          rob@example.com
Greg Sabino Mullane   greg@example.com
Matt Wade             no email
{{< /highlight >}}

导出为CVS格式：

{{< highlight python >}}
psql> COPY employee (name, email) TO '/home/smallfish/employee.csv' CSV HEADER;
{{< /highlight >}}

参考资料：[http://apress.com/book/view/9781590595473](http://apress.com/book/view/9781590595473)

