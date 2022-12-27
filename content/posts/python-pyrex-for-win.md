---
date: 2009-11-17
title: Win Python Pyrex 扩展

tags:
- Python

---

在偶的ubuntu里编写pyrex程序编译成so还是挺爽的，用 timeit.Timer 测试性能提升不少，今天在windows也尝试了一番。

需要的工具有：
- Pyrex [http://www.cosc.canterbury.ac.nz/greg.ewing/python/Pyrex/](http://www.cosc.canterbury.ac.nz/greg.ewing/python/Pyrex/)
- Dev C++ [http://www.bloodshed.net/devcpp.html](http://www.bloodshed.net/devcpp.html)

Pyrex 可以通过easy_install Pyrex来安装。
Dev C++ 安装完在系统环境变量Path里加上Dev C++安装目录/bin 目录。

测试扩展代码：

{{< highlight python >}}
# file: foo.pyx
""" simple pyrex module """
cdef class Foo:
    """ foo doc ... """
    cdef char *name
    def __init__(self, name):
        self.name = name
    def __repr__(self):
        return "foo names: %s" % (self.name)

# file: setup.py
from distutils.core import setup
from distutils.extension import Extension
from Pyrex.Distutils import build_ext

setup(
    name='foo', ext_modules=[Extension("foo", ["foo.pyx"])],
    cmdclass={'build_ext':build_ext}
)
{{< /highlight >}}

写好两个文件后，进入命令提示符：

{{< highlight python >}}
C:\>python setup.py build_ext --inplace -c mingw32
running build_ext
pyrexc foo.pyx --> foo.c
building 'foo' extension
creating build
creating build\temp.win32-2.5
creating build\temp.win32-2.5\Release
C:\Program Files\DEV-CPP\Bin\gcc.exe -mno-cygwin -mdll -O -Wall -ID:\Python25\include -ID:\Python25\PC -c foo.c -o build\temp.win32-2.5\Release\foo.o writing build\temp.win32-2.5\Release\foo.def C:\Program Files\DEV-CPP\Bin\dllwrap.exe -mno-cygwin -mdll -static --entry _DllMain@12 --output-lib build\temp.win32-2.5\Release\libfoo.a --def build\temp.win32-2.5\Release\foo.def -s build\temp.win32-2.5\Release\foo.o -LD:\Python25\libs -L D:\Python25\PCBuild -lpython25 -lmsvcr71 -o foo.pyd
{{< /highlight >}}

编译完毕，可以看到当前目录下多了：build目录、foo.c、foo.pyd。foo.pyd 即是编译好的二进制扩展。

{{< highlight python >}}
C:\>python
ActivePython 2.5.0.0 (ActiveState Software Inc.) based on
Python 2.5 (r25:51908, Mar 9 2007, 17:40:28) [MSC v.1310 32 bit (Intel)] on win32
Type "help", "copyright", "credits" or "license" for more information.
>>> import foo
<module 'foo' from 'foo.pyd'>
>>> foo.Foo("smallfish")
foo names: smallfish
{{< /highlight >}}


