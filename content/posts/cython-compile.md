---
date: 2009-11-19
title: Cython参考指南 - 编译
---

Cython代码跟Python不一样，必须要编译。

编译经过两个阶段：

* Cython编译.pyx文件为.c文件

* C编译器会把.c文件编译成.so文件(Windows上是.pyd)

以下会分节介绍几种方式来建立你的扩展模块。

注意： -a 选项，如果使用该选项将会为.c文件生成一份很漂亮的HTML文件，双击高亮的章节部分会展开代码，这对理解，优化和调试模块将会非常有帮助。

命令行

从命令行执行Cython编译器，输入选项和.pyx文件列表。

{{< highlight python >}}
$ cython -a yourmod.pyx
{{< /highlight >}}

会生成一个yourmod.c文件（指定-a选项会生成一个HTML文件）

编译.c文件取决于你的操作系统，请参考下如何在你的系统写Python扩展模块文档。

下面是一个Linux系统的例子：

{{< highlight python >}}
$ gcc -shared -pthread -fPIC -fwrapv -O2 -Wall -fno-strict-aliasing \
 -I/usr/include/python2.5 -o yourmod.so yourmod.c
{{< /highlight >}}

gcc需要提供包含的文件和扩展库的链接。

在目录里会生成yourmod.so文件。

现在只需要导入你的yourmod模块就可以了。

Distutils

确保你的系统已经安装好Distutils。

下面假设需要编译的文件叫hello.pyx。

建立一个setup.py的脚本：

{{< highlight python >}}
from distutils.core import setup
from distutils.extension import Extension
from Cython.Distutils import build_ext

ext_modules = [Extension("hello", ["hello.pyx"])]

setup(
    name = ’Hello world app’,
    cmdclass = {’build_ext’: build_ext},
    ext_modules = ext_modules
)
{{< /highlight >}}

在命令行执行：python setup.py build_ext --inplace

现在可以在shell或者脚本里正常导入使用了。

Pyximport

在纯Python代码里调用Cython代码：

{{< highlight python >}}
>>> import pyximport; pyximport.install()
>>> import helloworld
Hello World
{{< /highlight >}}

这仅仅是简单调用Cython，不需要C库也不需要构建脚本。

当然也可以实验性的在Python调用。允许在Python模块中运行Cython代码在任何一个.pyx和.py模块。这包
括标准库和包。如果Cython编译失败的话，pyximport会返回到加载失败的模块处。

.py安装是这样：

{{< highlight python >}}
>>> pyximport.install(pyimport = True)
{{< /highlight >}}

原文：[http://docs.cython.org/src/reference/compilation.html](http://docs.cython.org/src/reference/compilation.html)

