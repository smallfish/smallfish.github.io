---
date: 2010-06-29
title: 如何写 Go 代码？

tags:
- Golang
- Go

---

原文：[http://golang.org/doc/code.html](http://golang.org/doc/code.html)

简述

这篇文档描述了如何去写一个新的 package 和怎么去测试。本文假设你已经按照[http://golang.org/doc/install.html](http://golang.org/doc/install.html)装好Go。

在修改一个存在的 package 或者新建立一个package，确保先发一封邮件到 [http://groups.google.com/group/golang-nuts](http://groups.google.com/group/golang-nuts)，告诉大家你想做什么。这样有助于不要重复造轮子，在写代码之前最好讨论下。

社区资源

如果想获取实时帮助，可以加入 [http://freenode.net/](http://freenode.net/) 上 IRC 频道 #go-nuts。

Go 语言官方邮件列表是 [http://groups.google.com/group/golang-nuts](http://groups.google.com/group/golang-nuts).

Bugs 可以参考[http://code.google.com/p/go/issues/list](http://code.google.com/p/go/issues/list).

对于那些想尝试开发代码的用户，这里有另外一个邮件列表 [http://groups.google.com/group/golang-checkins](http://groups.google.com/group/golang-checkins)，邮件里包含了那些刚提交到 Go 代码库的消息。

建立Go包

下面的源码假设 package 的导入路径是 x/y，约定下保存的路径是：$GOROOT/src/pkg/x/y

Makefile

这将是很好的利用 Go-specific 工具里安排源码结构，如何按照顺序和构建代码。Go 使用 GUN make。所以首先在一个新的package 文件夹里建立一个 Makefile。最简单的做法就是从 [http://golang.org/src/pkg/container/vector/Makefile](http://golang.org/src/pkg/container/vector/Makefile) 源码包里拷贝一份。

{{< highlight python >}}
include ../../../Make.$(GOARCH)

TARG=container/vector
GOFILES=\
	intvector.go\
	stringvector.go\
	vector.go\

include ../../../Make.pkg
{{< /highlight >}}

当然在上面的源码包之外写一个新的 package ，通常的 Makefile 如下：

{{< highlight python >}}
include $(GOROOT)/src/Make.$(GOARCH)

TARG=mypackage
GOFILES=\
	my1.go\
	my2.go\

include $(GOROOT)/src/Make.pkg
{{< /highlight >}}

第一行 include 标准的定义和规则，package 的路径一般相对路径来代替 $(GOROOT)/src ，这样做到目的就是防止 make 时候 $(GOROOT) 含有空格，这样做很方便开发中使用Go。

TRAG 是包安装路径，这行就是其他程序导入到名字。在 Go 源码中，这个名字必须和文件夹中的 Makefile 中的一致，不需要 $GOROOT/src/pkg/ 前缀。在源码之外，你可以任意起名而不要和标准 package 的重名即可。常见的做法是独立一个 package 名：myname/tree, myname/filter，确保你的代码在 myname 里，运行 make install，将会编译后把二进制文件放入 $GOROOT/pkg，可以很方便的找到。

GOFILES 放入文件内需要编译的源码列表。多行用 \ 字符隔开。

如果想在 Go 源码树中新建包，需要添加列表到 $GOROOT/src/pkg/Makefile 中作为标准库编译，编译运行：

{{< highlight python >}}
cd $GOROOT/src/pkg
./deps.b
{{< /highlight >}}

更新依赖文件 Make.deps （这是每次 make all 或者 make build 都需要）

Go 源文件

在每个源码里第一行的名字应该是 Makefile 里面定义的 package 名，这里的 name 是作为默认的名字导入。（包里的每个 package 名都应该是同样的名字）Go的惯例是用路径的最后一个元素作为名字，比如 "crypto/rot13" 应该是 rot13。到目前为止，Go 是依靠 package 名字来确定一个二进制的文件，当然可能会很快的取消。

Go 是在一个包内编译源码文件，在一个文件定义常量，变量，类型或者函数，在其他文件内适用不需要再定义和声明。

想写一个干净而优美的 Go 代码不在本文范围。请参考[http://golang.org/doc/effective_go.html](http://golang.org/doc/effective_go.html)吧。

测试

Go 有一个轻量级的测试代码框架：gotest。编写一个测试代码，只需要新建一个跟你源码名字一样，加上：_test.go 即可，测试函数一般是 func TestXXX (t *testing.T)。每次运行测试其中的函数。如果想返回错误的话一般是返回： t.Error 或 t.Fail，测试时候就会抛错。详细 gotest 测试方法请参考：[http://golang.org/cmd/gotest/](http://golang.org/cmd/gotest/)

*_test.go 不需要在 Makefile 中声明。

运行测试，编译时候 make test 即可，和 gotest 效果一样。如果想单独测试某一个源码，比如：one_test.go，则运行：gotest one_test.go。

如果你的修改影响性能，可以添加一个性能测试函数（参考：[http://golang.org/cmd/gotest/](http://golang.org/cmd/gotest/)），然后运行：gotest -benchmarks=.

当你新的代码已经通过测试，也能运行，你就可以 review 和 commit.  [http://golang.org/doc/contribute.html](http://golang.org/doc/contribute.html)。

一个测试的例子

这是一个名字叫 numbers 测试 package ，定义了一个叫 Double 的函数，返回一个整型，结果是输入的参数乘以2。一共有三个文件。

第一个是 numbers.go

{{< highlight python >}}
package numbers

func Double(i int) int {
	return i * 2
}
{{< /highlight >}}

第二个是测试文件 numbers_test.go

{{< highlight python >}}
package numbers

import (
	"testing"
)

type doubleTest struct {
	in, out int
}

var doubleTests = []doubleTest{
	doubleTest{1, 2},
	doubleTest{2, 4},
	doubleTest{-5, -10},
}

func TestDouble(t *testing.T) {
	for _, dt := range doubleTests {
		v := Double(dt.in)
		if v != dt.out {
			t.Errorf("Double(%d) = %d, want %d.", dt.in, v, dt.out)
		}
	}
}
{{< /highlight >}}

最后是 Makefile 文件

{{< highlight python >}}
include $(GOROOT)/src/Make.$(GOARCH)

TARG=numbers
GOFILES=\
	numbers.go\

include $(GOROOT)/src/Make.
{{< /highlight >}}

运行 make install 将会构建和安装包到 GOROOT/pkg/ 文件夹内（他在系统的任何地方导入调用）。

运行 make test （或者 gotest）将会重新构建包，并会运行 numbers_test.go 中的 TestDouble 函数。如果输出 "PASS" 则表示测试成功通过。如果不是2到3的倍数，将会看到错误报告。

更详细的请查看[http://golang.org/cmd/gotest/](http://golang.org/cmd/gotest/)文档。

