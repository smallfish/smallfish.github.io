---
date: 2009-12-15
title: SSH Tunnel Memcached

tags:
- Linux
- SSH

---

最近一台服务器放进了移动机房，需要访问原电信机房一台Memcached服务器，Memcached服务是以内网形式启动。

依靠google大神，搜索出解决思路，在本地起一个SSH链接，通过本地一个端口实现对另外机器的映射或者叫做转发。

上周本来已经搞定，本周突发灵异事件，竟然不管了，最后百般尝试，完成结果如下：

移动机器IP：220.xxx.xxx.xxx 电信机器IP：155.xxx.xxx.xxx

在移动机器上执行：

{{< highlight python >}}
shell > ssh -N -f -L 11211:192.168.0.xxx:11211 root@155.xxx.xxx.xxx
{{< /highlight >}}

11211:192.168.0.xxx:11211，格式为：本地端口:memcache启动的IP:端口
这里没有用RSA认证，就直接输入密码。-N 是不需要shell，-f 是程序后台执行，其他参数参见ssh --help。

{{< highlight python >}}
shell > ps aux
{{< /highlight >}}

可以看见进程已经在了，下面开始测试代码。

{{< highlight python >}}

>>> import memcache
>>> mc = memcache.Client(['127.0.0.1:11211'],debug=True)
>>> print mc.get('name')
{{< /highlight >}}


