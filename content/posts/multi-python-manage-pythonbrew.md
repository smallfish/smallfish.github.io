---
date: 2011-04-03
title: pythonbrew - Python多版本管理利器
---

相信不少人在自己机器上有多个Python版本，我的机器上Python有四个版本：2.5.x、2.6.x、2.7和stackless。

测试Google App Engine时候需要切换到2.5，正式调试时候回归到2.6，自己玩的时候会选择2.7或者stackless，每次都是通过bash profile来调整，或者手动加link。真麻烦那。。。

无意间看到有一个Perl版本的brew工具：[http://search.cpan.org/~gugod/App-perlbrew-0.18/bin/perlbrew](http://search.cpan.org/~gugod/App-perlbrew-0.18/bin/perlbrew)！

安装：

{{< highlight python >}}
$ easy_install pythonbrew
$ pythonbrew_install

# 或手动下载
$ curl -kLO http://github.com/utahta/pythonbrew/raw/master/pythonbrew-install
$ chmod +x pythonbrew-install
$ ./pythonbrew-install
{{< /highlight >}}

把 source /xxx/.pythonbrew/etc/bashrc 加入到自己profile或者bashrc中去（xxx是自己的用户目录）

pythonbrew 常用命令如下：

install 安装版本：

{{< highlight python >}}
$ pythonbrew install 2.6.6
过程可以参考安装日志：~/.pythonbrew/log/build.log
如果最后看到make error失败，应该是test过程失败。可以采取：
$ pythonbrew install --force 2.6.6
{{< /highlight >}}

switch 选择版本：

{{< highlight python >}}
$ pythonbrew switch 2.6.6
{{< /highlight >}}

list 查看版本：

{{< highlight python >}}
$ pythonbrew list       # 列出目前已安装的版本
pythonbrew list -k  # 列出可以下载和安装的版本
{{< /highlight >}}

uninstall 卸载版本：

{{< highlight python >}}
$ pythonbrew uninstall 2.6.6
{{< /highlight >}}

参数还是很简单的。详见help或者[http://pypi.python.org/pypi/pythonbrew/](http://pypi.python.org/pypi/pythonbrew/)。

