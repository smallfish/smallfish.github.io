---
date: 2010-03-19
title: web.py 数据库操作指南
---

官网地址：[http://webpy.org/](http://webpy.org/)

web.py是一个小巧灵活的框架，最新稳定版是0.33。这里不介绍web开发部分，介绍下关于数据库的相关操作。

很多Pyer一开始都喜欢自己封装数据库操作类，本人亦如此。不过后来通过观摩web.py的源码，发现其数据库操作部分相当紧凑实用。推荐懒人可以尝试一下。

废话不多，先来安装，有两种方式：

1. easy_install方式，如果木有此工具，可以参考：[http://chenxiaoyu.org/blog/archives/23](http://chenxiaoyu.org/blog/archives/23)

{{< highlight python >}}
easy_install web.py
{{< /highlight >}}

2. 下载源码编译。地址： [http://webpy.org/static/web.py-0.33.tar.gz](http://webpy.org/static/web.py-0.33.tar.gz) ，解压后执行：

{{< highlight python >}}
python setup.py install
{{< /highlight >}}

web.py安装算到此结束，如果想使用其中的db功能，还得借助与相应数据库操作模块，比如MySQLdb、psycopg2。如果需要尝试连接池(database pool)功能，还得装下DBUtils。这几个模块都可以通过easy_install来安装。

下面开始使用吧！

1. 导入模块，定义数据库连接db。

{{< highlight python >}}
import web
db = web.database(dbn='postgres', db='mydata', user='dbuser', pw='')
{{< /highlight >}}

2. select 查询

{{< highlight python >}}
# 查询表
entries = db.select('mytable')

# where 条件
myvar = dict(name="Bob")
results = db.select('mytable', myvar, where="name = $name")
results = db.select('mytable', where="id>100")

# 查询具体列
results = db.select('mytable', what="id,name")

# order by
results = db.select('mytable', order="post_date DESC")

# group
results = db.select('mytable', group="color")

# limit
results = db.select('mytable', limit=10)

# offset
results = db.select('mytable', offset=10)
{{< /highlight >}}

3. 更新

{{< highlight python >}}
db.update('mytable', where="id = 10", value1 = "foo")
{{< /highlight >}}

4. 删除

{{< highlight python >}}
db.delete('mytable', where="id=10")
{{< /highlight >}}

5. 复杂查询

{{< highlight python >}}
# count
results = db.query("SELECT COUNT(*) AS total_users FROM users")
print results[0].total_users

# join
results = db.query("SELECT * FROM entries JOIN users WHERE entries.author_id = users.id")

# 防止SQL注入可以这么干
results = db.query("SELECT * FROM users WHERE id=$id", vars={'id':10})
{{< /highlight >}}

6 多数据库操作 (web.py大于0.3)

{{< highlight python >}}
db1 = web.database(dbn='mysql', db='dbname1', user='foo')
db2 = web.database(dbn='mysql', db='dbname2', user='foo')

print db1.select('foo', where='id=1')
print db2.select('bar', where='id=5')
{{< /highlight >}}

7. 事务

{{< highlight python >}}
t = db.transaction()
try:
    db.insert('person', name='foo')
    db.insert('person', name='bar')
except:
    t.rollback()
    raise
else:
    t.commit()

# Python 2.5+ 可以用with
from __future__ import with_statement
with db.transaction():
    db.insert('person', name='foo')
    db.insert('person', name='bar')
{{< /highlight >}}


