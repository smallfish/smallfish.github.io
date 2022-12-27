---
date: 2009-12-16
title: 修改 ModPython 下 PYTHON_EGG_CACHE 报错
---

环境：Linux Apache Python(mod_python)

换了一台新机器，没有配置Mod_Python了，在一些应用里import MySQLdb出现了下面错误：

{{< highlight python >}}
ExtractionError: Can't extract file(s) to egg cache
The following error occurred while trying to extract file(s) to the Python egg
cache:
  [Errno 13] Permission denied: '/root/.python-eggs'
The Python egg cache directory is currently set to:
  /root/.python-eggs
Perhaps your account does not have write access to this directory?  You can
change the cache directory by setting the PYTHON_EGG_CACHE environment
variable to point to an accessible directory.
{{< /highlight >}}

解决办法有两种：

1.设置PYTHON_EGG_CACHE环境变量

{{< highlight python >}}
$ SetEnv PYTHON_EGG_CACHE /tmp/aaa/
{{< /highlight >}}

目录权限注意要是apache用户，或者简单点就777

2.把egg格式转成目录

{{< highlight python >}}
$ cd /python-path/site-packages/
$ mv MySQL_python-1.2.3c1-py2.5-linux-x86_64.egg foo.zip
$ mkdir MySQL_python-1.2.3c1-py2.5-linux-x86_64.egg
$ cd MySQL_python-1.2.3c1-py2.5-linux-x86_64.egg
$ unzip ../foo.zip
$ rm ../foo.zip
{{< /highlight >}}


