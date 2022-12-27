---
date: 2010-05-30
title: Cython 教程 - 调用外部C语言函数

tags:
- Python
- Cython

---

一般情况完全可以在 Python 里导入 from math import sin 然后调用 sin() 函数。然而，调用C里面的 sin() 函数速度会更快，尤其在复杂的循环里。在 Cython 里可以这样声明和使用：

{{< highlight python >}}
cdef extern from "math.h":
    double sin(double)

cdef double f(double x):
    return sin(x*x)
{{< /highlight >}}

请注意，上面的代码声明了 math.h 里的函数，提供给 Cython 使用。C编译器在编译时将会看到 math.h 的声明，但 Cython 不会去分析 math.h 和单独的定义。

当调用一个C函数时，一定要注意引入适当的链接库。这个依赖于特定的平台；下面的例子可以在Linux和Mac OS X下运行：

{{< highlight python >}}
from distutils.core import setup
from distutils.extension import Extension
from Cython.Distutils import build_ext

ext_modules=[
    Extension("demo",
              ["demo.pyx"],
              libraries=["m"]) # Unix-like specific
]

setup(
  name = "Demos",
  cmdclass = {"build_ext": build_ext},
  ext_modules = ext_modules
)
{{< /highlight >}}

跟从 math 库里使用 sin() 函数一样，它可以声明和导入任何C库，Cython会生成正确的共享或者静态链接库。

参考：[http://docs.cython.org/src/tutorial/external.html](http://docs.cython.org/src/tutorial/external.html)

