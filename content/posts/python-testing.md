---
date: 2013-09-12
title: Python Testing

tags:
- Python

---

代码写多了越发觉得测试的重要性，之前一直喜欢“目测”的做法已经不值得推荐了。当然，这只是一个玩笑。

在Python代码里测试大概有这么几种：[doctest](http://docs.python.org/2/library/doctest.html)、[unittest](http://docs.python.org/2/library/unittest.html)和[nose](http://nose.readthedocs.org/en/latest/)（第三方工具）。

个人推荐nose，简单的话doctest也已经足够了。

首先：

* 代码要易于测试，代码写完对应的测试应该配套跟上。
* 测试要简单，轻便。

先说最简单的doctest吧，顾名思义，doctest就是在文档（docstring）里完成测试。看以下例子（/tmp/1.py）：

{{< highlight python >}}
def add(a, b):
    """
    >>> add(10, 20)
    30
    """
    return a + b

if __name__ == '__main__':
    import doctest
    doctest.testmod()
{{< /highlight >}}
    
可以看出doctest是不是很简单？是不是想起了普通解释器（>>>），运行一下测试（注意-v选项）：

{{< highlight bash >}}
smallfish@debian:~$ python /tmp/1.py -v
Trying:
    add(10, 20)
Expecting:
    30
ok
1 items had no tests:
    __main__
1 items passed all tests:
   1 tests in __main__.add
1 tests in 2 items.
1 passed and 0 failed.
Test passed.
{{< /highlight >}}

作为简单的模块或者工具，main里实现测试已经足够了。但是稍大一些工程或应用，doctest太多的话，看上去会觉得有点臃肿。

下面介绍下nose，偷偷的说下，nose是支持doctest、unittest测试的，所以嘛。先安装一下：

{{< highlight bash >}}
smallfish@debian:~$ pip install nose
{{< /highlight >}}

继续跑一下上面的例子：

{{< highlight bash >}}
smallfish@debian:~$ nosetests /tmp/1.py --with-doctest -v
Doctest: 1.add ... ok

----------------------------------------------------------------------
Ran 1 test in 0.006s

OK
{{< /highlight >}}

好吧，写点常见的例子，目录结构如下（foo模块以及foo的测试代码）：

{{< highlight bash >}}
smallfish@debian:~$ tree /tmp/foomodule/
/tmp/foomodule/
|-- foo
|   |-- a.py
|   |-- b.py
|   `-- __init__.py
`-- tests
    |-- test_a.py
    `-- test_b.py
{{< /highlight >}}

模块代码如下：

{{< highlight python >}}
# /tmp/foomodule/foo/a.py 
def add(a, b):
    return a + b

def double(a):
    return a * 2
    
# /tmp/foomodule/foo/b.py 
import memcache

class Cache:

    def __init__(self, server):
        self.cache = memcache.Client([server])

    def get(self, name):
        return self.cache.get(name)

    def set(self, name, value):
        return self.cache.set(name, value)

    def delete(self, name):
        return self.cache.delete(name)

    def close(self):
        self.cache.disconnect_all()
{{< /highlight >}}


对应的测试代码：

{{< highlight python >}}
# /tmp/foomodule/tests/test_a.py 
from foo.a import add, double

def test_add():
    v = add(10, 20)
    assert v == 30

def test_double():
    v = double(10)
    assert v == 20

# /tmp/foomodule/tests/test_b.py 
from foo.b import Cache

class TestCache:

    def setUp(self):
        self.cache = Cache("127.0.0.1:11211")
        self.key   = "name"
        self.value = "smallfish"

    def tearDown(self):
        self.cache.close()

    def test_00_get(self):
        v = self.cache.get(self.key)
        assert v == None

    def test_01_set(self):
        v = self.cache.set(self.key, self.value)
        assert v == True
        v = self.cache.get(self.key)
        assert v == self.value

    def test_02_delete(self):
        v = self.cache.delete(self.key)
        assert v == True
{{< /highlight >}}

先不解释，跑一跑（-v -s选项，针对当前路径下tests目录测试）：

{{< highlight bash >}}
smallfish@debian:/tmp/foomodule$ nosetests -s -v
test_a.test_add ... ok
test_a.test_double ... ok
test_b.TestCache.test_00_get ... ok
test_b.TestCache.test_01_set ... ok
test_b.TestCache.test_02_delete ... ok

----------------------------------------------------------------------
Ran 5 tests in 0.024s

OK
{{< /highlight >}}

简单吧？nose默认是当前目录下tests目录进行测试，默认规则：文件（含有test），函数（test_开头），类（Test开头）。

nose完整文档请参考：[http://nose.readthedocs.org/en/latest/](http://nose.readthedocs.org/en/latest/)





