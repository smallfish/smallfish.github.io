---
date: 2014-11-06
title: 翻译-如何组织Go代码
---

原文地址：[http://talks.golang.org/2014/organizeio.slide](http://talks.golang.org/2014/organizeio.slide)，译文尽量贴近原文，会适当的增删，勿拍。

包

Go 程序都是由包构成，每个文件都以 `package` 开头，程序主体执行从 `main` 包开始：

{{< highlight bash >}}
package main

import "fmt"

func main() {
        fmt.Println("Hello, world!")
}
{{< /highlight >}}

最简单的一个 Go 程序，只需要写一个 `main` 包即可。

`hello world` 第二行导入了 `fmt` 包，第四行 `Println` 是 `fmt` 包里的公开导出的函数。

示例包：`fmt`

{{< highlight bash >}}
// Package fmt implements formatted I/O.
package fmt

// Println formats using the default formats for its
// operands and writes to standard output.
func Println(a ...interface{}) (n int, err error) {
        ...
}

func newPrinter() *pp {
        ...
}
{{< /highlight >}}

其中 `Println` 是公开（`public`）函数，只需要第一个字母大写，在其他模块内都是公开可见的（区别于私有 `private` 函数）。

另外一个函数 `newPrinter` 则是私有函数，以小写字母开头，这样的函数只能在模块（`fmt`）内使用。

通过包来组织和区分相关代码，包可大可小，也可以由多个文件构成，这些文件需要在一个目录内。

Go 源码 `net/http` 包导出 100 个命名（共 18 个文件），而 `errors` 包只导出 1 个名字（只有 1 文件）。

如何给包命名？

取名尽量简单和可阅读，不要使用下划线：

* io/ioutil，不要用 io/util
* suffixarray，不要用 suffix_array

也不要太笼统，比如 `util` 意思很模糊。

包名也是类型和函数的一部分，比如：

{{< highlight bash >}}
buf := new(bytes.Buffer)
{{< /highlight >}}

这里不要取名为 `bytes.BytesBuffer`，过于累赘。

选择一个好的命名真的很重要，大部分时候我们写代码都在为取名而烦恼啊。。。

包的测试

测试文件和源码同级目录，文件名都是 `_test.go` 结尾：

{{< highlight bash >}}
package fmt

import "testing"

var fmtTests = []fmtTest{
        {"%d", 12345, "12345"},
        {"%v", 12345, "12345"},
        {"%t", true, "true"},
}

func TestSprintf(t *testing.T) {
        for _, tt := range fmtTests {
            if s := Sprintf(tt.fmt, tt.val); s != tt.out {
                    t.Errorf("...")
            }
        }
}
{{< /highlight >}}

差不多了，建议一个源码文件对应一个测试文件。

代码组织

工作目录（workspace）

新版 Go 依赖 `$GOPATH` 这个环境变量来区分工作目录。代码放在同一个 `workspace` 内，可以包含自己的包或远程仓库（如： `git` 或 `hg`），用过 `go get` 应该都知道。

Go 相关工具可以很容易的区分出工作空间，构建时候不需要依赖 `Makefile` 或 `build.xml` 类似文件 ，按照目录划分好就可以啦。

比如：

{{< highlight bash >}}
$GOPATH/
    src/
       github.com/user/repo/
           mypkg/
              mysrc1.go
              mysrc2.go
           cmd/mycmd/
                main.go
   bin/
       mycmd
{{< /highlight >}}

这里示例一下工作目录吧

{{< highlight bash >}}
mkdir /tmp/gows
GOPATH=/tmp/gows
{{< /highlight >}}

`$GOPATH` 环境变量前面以及提到，后续的安装和构建包都依赖这个：

{{< highlight bash >}}
$ go get github.com/dsymonds/fixhub/cmd/fixhub
{{< /highlight >}}

`go get` 命令则会远程仓库下载源码到自己的工作目录内（需要相关的版本工具，比如：`git`）

`go install` 命令则可以编译和分发文件到 `$GOPATH/bin/fixhub` 位置。

我的工作目录大概如下：

{{< highlight bash >}}
$GOPATH/
    bin/fixhub                              # installed binary
    pkg/darwin_amd64/                       # compiled archives
        code.google.com/p/goauth2/oauth.a
        github.com/...
    src/                                    # source repositories
        code.google.com/p/goauth2/
            .hg
            oauth                           # used by package go-github
            ...
        github.com/
            golang/lint/...                 # used by package fixhub
                .git
            google/go-github/...            # used by package fixhub
                .git
            dsymonds/fixhub/
                .git
                client.go
                cmd/fixhub/fixhub.go        # package main
{{< /highlight >}}

可以看到下载了很多远程的仓库，编译后可执行文件是 `bin/fixhub`。

为什么要划分目录结构呢？

通过目录结构来区分，可以免去配置的麻烦，不像其他语言会依赖 `Makefile` 或 `build.xml` 文件。

减少配置的时间，可以更多的时间去码字，另外大部分用户目录结构都类似，这样也更有利于去分享代码。

有时候我们可能有很多的工作目录，但是大部分人只用一个。有个建议这样来声明 `GOPATH`：

{{< highlight bash >}}
GOPATH=$HOME
{{< /highlight >}}

这样 `src`，`bin` 和 `pgk` 都会在你的用户 `home` 目录下了。

这里的建议应该是针对一台机器上多个用户来说吧，一般来说 ` $HOME/bin` 可能在 `PATH` 中了，调用起来更为方便一些。

快速切换目录

{{< highlight bash >}}
CDPATH=$GOPATH/src/github.com:$GOPATH/src/code.google.com/p

$ cd dsymonds/fixhub
/tmp/gows/src/github.com/dsymonds/fixhub
$ cd goauth2
/tmp/gows/src/code.google.com/p/goauth2
$
{{< /highlight >}}

还可以写个函数放入到 `~/.profile` 内：

{{< highlight bash >}}
gocd () { cd `go list -f '{{.Dir}}' $1` }
{{< /highlight >}}
	
这样移动更方便一些：

{{< highlight bash >}}
$ gocd .../lint
/tmp/gows/src/github.com/golang/lint
$
{{< /highlight >}}

`go` 工具有很多功能：

{{< highlight bash >}}
$ go help
Go is a tool for managing Go source code.

Usage:

go command [arguments]

The commands are:
{{< /highlight >}}

常用的功能参数如：

{{< highlight bash >}}
build       compile packages and dependencies
get         download and install packages and dependencies
install     compile and install packages and dependencies
test        test packages
{{< /highlight >}}

还有一些其他有用的功能参数，比如：`vet` 和 `fmt`，后者是大招啊。

依赖管理

默认情况下 `go get` 都会去下载最新的代码然后构建，除非被中断。

在开发环境下这个没什么影响，但肯定不适用于生产环境。

略去数十字（`vendoring` 啥意思..），不喜欢这种做法，推荐 [gopkg.in](http://gopkg.in)：

{{< highlight bash >}}
gopkg.in/user/pkg.v3 -> github.com/user/pkg (branch/tag v3, v3.N, or v.3.N.M)
{{< /highlight >}}

命名

程序其实就是一堆名字的构成，取名还是挺花时间和成本的。

简单来说，长的名字浪费空间，而且在可读性方面很重要，好的名字一眼就能看意图。

多花点时间，取一些简短和可理解的名字吧。

命名风格

* 使用驼峰 `camelCase`，而不是下划线 `_underscores` 连接。
* 局部变量尽量短，短，短，1-2 个字符的情况很常见。
* 包的名字一般来说，都是小写字母。
* 全局（公开）变量

不建议这样：

* bytes.Buffer，不要 bytes.ByteBuffer
* zip.Reader，不要 zip.ZipReader
* errors.New，不要 errors.NewError
* r 不要用在 bytesReader
* i 不要用在 loopIterator

文档

文档位置在模块或者导出的名字前面：

{{< highlight bash >}}
// Join concatenates the elements of elem to create a single string.
// The separator string sep is placed between elements in the resulting string.
func Join(elem []string, sep string) string {
{{< /highlight >}}
	
通过 `godoc` 在 web 查看该函数时候，文档位置位于函数原型下方（图略去）。

请用英语书写文档句子和段落（无视我大汉字啊）

{{< highlight bash >}}
// Join concatenates…         good
// This function…             bad
{{< /highlight >}}

包文档位于模块最上面：

{{< highlight bash >}}
	// Package fmt…
	package fmt	
{{< /highlight >}}

关于 Go docs 请参考：[http://talks.golang.org/2014/godoc.org](http://talks.golang.org/2014/godoc.org)

最后，这是一份演讲稿，相对来说内容偏少，随意看看吧。。



