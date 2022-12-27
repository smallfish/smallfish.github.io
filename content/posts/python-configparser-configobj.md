---
date: 2010-04-19
title: Python ConfigParser 与 ConfigObj INI 配置读写顺序

tags:
- Python

---

默认的ConfigParser对于选项是按照字母顺序排列的。如下代码：

{{< highlight python >}}
>>> from ConfigParser import ConfigParser
>>> cf = ConfigParser()
>>> cf.add_section('d')
>>> cf.set('d', 'name', 'smallfish')
>>> cf.add_section('a')
>>> cf.set('a', 'name', 'smallfish2')
>>> cf.write(open('d:/a.ini', 'w'))
>>> cf = None
{{< /highlight >}}

生成配置如下：

{{< highlight python >}}
[a]
name = smallfish2
[d]
name = smallfish
{{< /highlight >}}

翻阅了官方文档似乎对ConfigParser中section的顺序没啥解说，毕竟字典本身就是无序的，如果想修改估计只能从源码入手把。不过有一个ConfigObj库还不错，可以实现顺序，当然功能不仅仅如此啦。下载地址：[http://www.voidspace.org.uk/python/configobj.html](http://www.voidspace.org.uk/python/configobj.html)

代码片段如下：

{{< highlight python >}}
>>> from configobj import ConfigObj
>>> config = ConfigObj()
>>> config.filename = 'd:/a.ini'
>>> config['d'] = {}
>>> config['d']['name'] = 'smallfish'
>>> config['a'] = {}
>>> config['a']['name'] = 'smallfish2'
>>> config.write()
{{< /highlight >}}

生成配置如下：

{{< highlight python >}}
[d]
name = smallfish
[a]
name = smallfish2
{{< /highlight >}}


