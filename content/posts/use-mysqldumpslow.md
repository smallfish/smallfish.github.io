---
date: 2009-12-15
title: mysqldumpslow 慢查询日志分析工具

tags:
- MySQL

---

mysql自带的这个玩意挺好使的，可以对慢查询里的sql进行排序、计算等操作。

首先得配置my.cnf：

{{< highlight python >}}

log_slow_queries = /path/slow.log # 定义log位置，注意要有写入的权限

{{< /highlight >}}

具体的使法如下：

{{< highlight python >}}

mysqldumpslow -s c -t 40 /path/slow.log

{{< /highlight >}}

出来的结果是访问次数最多的40个sql，几个参数大概意思如下：

{{< highlight python >}}

-t 显示多少条
-s 排序，默认是at。c是次数，t是时间，l是lock时间，r是返回结果。如果是ac，at，al，ar则是倒序
-g 可以用正则匹配部分语句

{{< /highlight >}}

可以参考mysqldumpslow --help，通过这个工具可以看到哪些锁表，或者其他性能问题，还能看到某些SQL_NO_CACHE提示呢，去想办法优化把！

