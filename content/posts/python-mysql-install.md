---
date: 2009-11-10
title: Python MySQL 库安装笔记
---

其实MySQL-python安装很简直，以前一直也没在意，今天发觉换了1.2.3新版本，ms蹦出很多问题来了。
做个记录，防止以后有问题无处可查。

一般步骤是：

1. 安装easy_install

{{< highlight python >}}
shell > wget http://peak.telecommunity.com/dist/ez_setup.py
shell > python ez_setup.py
{{< /highlight >}}

会自动根据本机的py版本选择对应的egg，安装完可以看到有/usr/bin/easy_install程序了

2. 安装MySQL-python

{{< highlight python >}}
shell > easy_install MySQL-python
{{< /highlight >}}

到这里安装算是完成了，不过接下来测试就郁闷了。

在import MySQLdb出现了两个错误：

{{< highlight python >}}
a). ImportError: libmysqlclient_r.so.15: cannot open shared object file: No such file or directory
{{< /highlight >}}

这个错误一般解决比较简单，把路径加入到LD_LIBRARY_PATH即可，不过偶的现象比较强，因为没装MySQL，哈哈

{{< highlight python >}}
b). ImportError: /lib/tls/libc.so.6: version `GLIBC_2.4' not found
{{< /highlight >}}

解决这个错误的办法是不用easy_install了，直接下载MySQL-python-1.2.2.tar.gz包，然后就是三步走：

{{< highlight python >}}
shell > tar zxvf MySQL-python-1.2.2.tar.gz
shell > cd MySQL-python-1.2.2
shell > python setup.py install
{{< /highlight >}}


