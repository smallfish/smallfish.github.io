---
date: 2009-11-10
title: Python MySQLdb 查询返回字典结构
---

MySQLdb默认查询结果都是返回tuple，输出时候不是很方便，必须按照0，1这样读取，无意中在网上找到简单的修改方法，就是传递一个cursors.DictCursor就行。

默认程序：

{{< highlight python >}}
import MySQLdb
db  = MySQLdb.connect(host='localhost', user='root', passwd='123456', db='test')
cur = db.cursor()
cur.execute('select * from user')
rs = cur.fetchall()
print rs
# 返回类似如下
# ((1000L, 0L), (2000L, 0L), (3000L, 0L))
{{< /highlight >}}

修改后：

{{< highlight python >}}
import MySQLdb
import MySQLdb.cursors
db = MySQLdb.connect(host='localhost', user='root', passwd='123456', db='test',
                     cursorclass=MySQLdb.cursors.DictCursor)
cur = db.cursor()
cur.execute('select * from user')
rs = cur.fetchall()
print rs
# 返回类似如下
# ({'age': 0L, 'num': 1000L}, {'age': 0L, 'num': 2000L}, {'age': 0L, 'num': 3000L})
{{< /highlight >}}

或者也可以用下面替换connect和cursor部分

{{< highlight python >}}
db  = MySQLdb.connect(host='localhost', user='root', passwd='123456', db='test')
cur = db.cursor(cursorclass=MySQLdb.cursors.DictCursor)
{{< /highlight >}}


