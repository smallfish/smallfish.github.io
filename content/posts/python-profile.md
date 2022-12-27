---
date: 2013-08-28
title: Python Profile 工具性能分析

tags:
- Python

---

最近碰到“程序速度大大降低”的说法，还是直接用数据说明比较有信服力，以及可以找出真正问题所在。

Python自带了几个性能分析的模块：profile、cProfile和hotshot，使用方法基本都差不多，无非模块是纯Python还是用C写的。

官网文档参考：[http://docs.python.org/2/library/profile.html](http://docs.python.org/2/library/profile.html)

本文示例基于cProfile模块，先写点测试代码（test1.py）：

{{< highlight python >}}
import time

def func1():
    sum = 0
    for i in range(1000000):
        sum += i

def func2():
    time.sleep(10)

func1()
func2()
{{< /highlight >}}

运行cProfile命令如下：

{{< highlight bash >}}
$ python -m cProfile -o test1.out test1.py
{{< /highlight >}}

这里是以模块方式直接保存profile结果，当然也可以在程序中引入cProfile模块。

好了，上面的测试程序大概运行10秒左右，可以看下最终的性能分析数据。

{{< highlight bash >}}
$ python -c "import pstats; p=pstats.Stats('test1.out'); p.print_stats()"

Wed Aug 28 22:19:45 2013    test1.out

         6 function calls in 10.163 seconds

   Random listing order was used

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.024    0.024   10.163   10.163 test1.py:1(<module>)
        1   10.001   10.001   10.001   10.001 {time.sleep}
        1    0.092    0.092    0.138    0.138 test1.py:3(func1)
        1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
        1    0.000    0.000   10.001   10.001 test1.py:8(func2)
        1    0.046    0.046    0.046    0.046 {range}
{{< /highlight >}}

分析的数据还很清晰和简单的吧，大概能看出几列数据的意思，重点关注下时间和函数调用次数，再来排个序：

{{< highlight bash >}}
$ python -c "import pstats; p=pstats.Stats('test1.out'); p.sort_stats('time').print_stats()"

Wed Aug 28 22:19:45 2013    test1.out

         6 function calls in 10.163 seconds

   Ordered by: internal time

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1   10.001   10.001   10.001   10.001 {time.sleep}
        1    0.092    0.092    0.138    0.138 test1.py:3(func1)
        1    0.046    0.046    0.046    0.046 {range}
        1    0.024    0.024   10.163   10.163 test1.py:1(<module>)
        1    0.000    0.000   10.001   10.001 test1.py:8(func2)
        1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
{{< /highlight >}}

可以看出测试程序中最耗费时间点是：time.sleep和range。

sort_stats支持一下参数：

{{< highlight bash >}}
calls, cumulative, file, line, module, name, nfl, pcalls, stdname, time
{{< /highlight >}}

pstats模块还支持交互式：

{{< highlight bash >}}
$ python -m pstats test1.out
Welcome to the profile statistics browser.
test1.out% ### help 或者输入字母按下tab，可以补全的
test1.out% stats 10
{{< /highlight >}}

这里似乎告一段落了，只是输出式的性能分析似乎有点乏味，开源世界这么美好，肯定会有些更美观和帅气的方法吧。

一步步来，先安装依赖项（请选择对应系统平台）：

* [gprof2dot.py](http://gprof2dot.jrfonseca.googlecode.com/git/gprof2dot.py)
* [graphviz](http://www.graphviz.org/Download.php)

PS：Ubuntu系统请注意请注意，pstats模块因为某些原因不在Python包中，需要单独安装：python-profiler包

装好依赖后，如此这番：

{{< highlight bash >}}
$ ./gprof2dot.py -f pstats test1.out | dot -Tpng -o test1.png
{{< /highlight >}}

看看test1.png，一切都了然于心了吧。。。

![python profile](/images/python-profile.png)



