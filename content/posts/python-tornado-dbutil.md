---
date: 2009-12-01
title: tornado.database添加PooledDB连接池功能

tags:
- Python

---

tornado.database模块简单包装了下对MySQL的操作，短小精悍。

无奈源码中无连接池功能，遂加上了一段DBUtils模块功能。

主要修改了reconnect()方法，大致在database.py第86行左右。(tornado 0.2 win版)

原代码如下：

{{< highlight python >}}
    def reconnect(self):
        """Closes the existing database connection and re-opens it."""
        self.close()
        self._db = MySQLdb.connect(**self._db_args)
        self._db.autocommit(True)
{{< /highlight >}}

修改后：

{{< highlight python >}}
    def reconnect(self):
        """Closes the existing database connection and re-opens it."""
        self.close()
        try:
            from DBUtils import PooledDB
            pool_con = PooledDB.PooledDB(creator=MySQLdb, **self._db_args)
            self._db = pool_con.connection()
        except:
            self._db = MySQLdb.connect(**self._db_args)
            self._db.autocommit(True)
{{< /highlight >}}

至于安装DBUtils模块可以去[http://pypi.python.org/pypi/DBUtils/](http://pypi.python.org/pypi/DBUtils/)下载，也可以简单的用easy_install：

{{< highlight python >}}
easy_install -U DBUtils
{{< /highlight >}}

PooledDB有这么几个参数：

{{< highlight python >}}

* creator
    可以生成 DB-API 2 连接的任何函数或 DB-API 2 兼容的数据库连接模块。
* mincached
    启动时开启的空连接数量(缺省值 0 意味着开始时不创建连接)
* maxcached
    连接池使用的最多连接数量(缺省值 0 代表不限制连接池大小)
* maxshared
    最大允许的共享连接数量(缺省值 0 代表所有连接都是专用的)
* maxconnections
    最大允许连接数量(缺省值 0 代表不限制)
* blocking
    设置在达到最大数量时的行为(缺省值 0 或 False)
* maxusage
    单个连接的最大允许复用次数(缺省值 0 或 False 代表不限制的复用)
* setsession:
    一个可选的SQL命令列表用于准备每个会话，如 ["set datestyle to german", ...]

{{< /highlight >}}


creator 函数或可以生成连接的函数可以接受这里传入的其他参数，例如主机名、数据库、用户名、密码等。你还可以选择传入creator函数的其他参数，允许失败重连和负载均衡。

具体可以参照下：[http://www.webwareforpython.org/DBUtils/Docs/UsersGuide.zh.html](http://www.webwareforpython.org/DBUtils/Docs/UsersGuide.zh.html)

