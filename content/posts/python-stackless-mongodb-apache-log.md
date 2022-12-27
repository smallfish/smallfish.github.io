---
date: 2010-03-04
title: Python(Stackless) + MongoDB Apache 日志(2G)分析
---

为何选择Stackless？ [http://www.stackless.com](http://www.stackless.com)

Stackless可以简单的认为是Python一个增强版，最吸引眼球的非“微线程”莫属。微线程是轻量级的线程，与线程相比切换消耗的资源更小，线程内共享数据更加便捷。相比多线程代码更加简洁和可读。此项目是由EVE Online推出，在并发和性能上确实很强劲。安装和Python一样，可以考虑替换原系统Python。:)

为何选择MongoDB？ [http://www.mongodb.org](http://www.mongodb.org)

可以在官网看到很多流行的应用采用MongoDB，比如sourceforge，github等。相比RDBMS有啥优势？首先在速度和性能上优势最为明显，不仅可以当作类似KeyValue数据库来使，还包含了一些数据库查询（Distinct、Group、随机、索引等特性）。再有一点特性就是：简单。不论是应用还是文档，还是第三方API，几乎略过一下就可以使用。不过有点遗憾的就是，存储的数据文件很大，超过正常数据的2-4倍之间。本文测试的Apache日志大小是2G，生产的数据文件有6G。寒...希望在新版里能有所缩身，当然这个也是明显的以空间换速度的后果。

本文除去上面提及到的两个软件，还需要安装pymongo模块。[http://api.mongodb.org/python/](http://api.mongodb.org/python/)

模块安装方式有源码编译和easy_install，这里就不再累赘。

1. 从Apache日志中分析出需要保存的资料，比如IP，时间，GET/POST，返回状态码等。

{{< highlight python >}}
fmt_str  = '(?P<ip>[.\d]+) - - \[(?P<time>.*?)\] "(?P<method>.*?) (?P<uri>.*?) HTTP/1.\d" (?P<status>\d+) (?P<length>.*?) "(?P<referere>.*?)" "(?P<agent>.*?)"'
fmt_name = re.findall('\?P<(.*?)>', fmt_str)
fmt_re   = re.compile(fmt_str)
{{< /highlight >}}

定义了一个正则用于提取每行日志的内容。fmt_name就是提取尖括号中间的变量名。

2. 定义MongoDB相关变量，包括需要存到collection名称。Connection采取的是默认Host和端口。

{{< highlight python >}}
conn     = Connection()
apache   = conn.apache
logs     = apache.logs
{{< /highlight >}}

3. 保存日志行

{{< highlight python >}}
def make_line(line):
    m = fmt_re.search(line)
    if m:
        logs.insert(dict(zip(fmt_name, m.groups())))
{{< /highlight >}}

4. 读取Apache日志文件

{{< highlight python >}}
def make_log(log_path):
    with open(log_path) as fp:
        for line in fp:
            make_line(line.strip())
{{< /highlight >}}

5. 运行把。

{{< highlight python >}}
if __name__ == '__main__':
    make_log('d:/apachelog.txt')
{{< /highlight >}}

脚本大致情况如此，这里没有放上stackless部分代码，可以参考下面代码：

{{< highlight python >}}
import stackless
def print_x(x):
    print x
stackless.tasklet(print_x)('one')
stackless.tasklet(print_x)('two')
stackless.run()
{{< /highlight >}}

tasklet操作只是把类似操作放入队列中，run才是真正的运行。这里主要用于替换原有多线程threading并行分析多个日志的行为。

补充：

Apache日志大小是2G，671万行左右。生成的数据库有6G。

硬件：Intel(R) Core(TM)2 Duo CPU E7500 @ 2.93GHz 台式机

系统：RHEL 5.2 文件系统ext3

其他：Stackless 2.6.4 MongoDB 1.2

在保存300万左右时候，一切正常。不管是CPU还是内存，以及插入速度都很不错，大概有8-9000条/秒。和以前笔记本上测试结果基本一致。再往以后，内存消耗有点飙升，插入速度也降低。500万左右记录时候CPU达到40%，内存消耗2.1G。在生成第二个2G数据文件时候似乎速度和效率又提升上去了。最终保存的结果不是太满意。

后加用笔记本重新测试了一下1000万数据，速度比上面的671万明显提升很多。初步怀疑有两个地方可能会影响性能和速度：

1. 文件系统的差异。笔记本是Ubuntu 9.10，ext4系统。搜了下ext3和ext4在大文件读写上会有所差距。

2. 正则匹配上。单行操作都是匹配提取。大文件上应该还有优化的空间。

