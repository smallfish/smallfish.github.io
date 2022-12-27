---
date: 2012-03-14
title: 如何编写 Go 程序（最新版）

tags:
- Golang
- Go

---

官网文档（weekly）：[http://weekly.golang.org/doc/code.html](http://weekly.golang.org/doc/code.html)

中文版（旧版）：[http://chenxiaoyu.org/2010/06/29/how-to-write-go.html](http://chenxiaoyu.org/2010/06/29/how-to-write-go.html)

本想翻译下官网版本，翻译起来觉得有点麻烦和啰嗦。还是自己摘抄一些片段，适当删减。

本文文档基于 weekly.2012-03-13 版本，应该是传闻中的 Go1 RC1 版本。

鉴于新版多了 go 命令，整合了以前旧版中的不少命令，比如： 6g/6l 或者采用 Makefile 之类编译的东西。

可以自己试试下：

{{< highlight bash >}}

$ go

Go is a tool for managing Go source code.
Usage:
        go command [arguments]
The commands are:
    build       compile packages and dependencies
    clean       remove object files
    doc         run godoc on package sources
    env         print Go environment information
 … 略去若干字

{{< /highlight >}}


言归正传，开始进入正题。 go 命令依赖一个重要的环境变量：**$GOPATH**

在类似 Unix 环境大概这样设置：

{{< highlight bash >}}
GOPATH=/home/user/ext:/home/user/mygo
{{< /highlight >}}

Windows 系统请注意分隔符，是分号（非冒号）。

以上 **$GOPATH** 目录约定有三个子目录：

* src  存放源代码（比如：.go .c .h .s等）
* pkg  编译后生成的文件（比如：.a）
* bin  编译后生成的可执行文件（为了方便，可以把此目录加入到 $PATH 变量中）

下面来一个实例来说明，文件和目录大概如下：

a). 设置 $GOPATH，写入 $HOME/.profile 以下内容

{{< highlight bash >}}
export GOPATH=$HOME/mygo
export PATH=$PATH:$HOME/mygo/bin
{{< /highlight >}}

b). 建立包和目录：$GOPATH/src/example/newmath/sqrt.go（包名："example/newmath"）

{{< highlight bash >}}
$ mkdir -p $GOPATH/src/example
$ cd $GOPATH/src/example
$ mkdir newmath

// $GOPATH/src/example/newmath/sqrt.go源码如下：
package newmath

func Sqrt(x float64) float64 {
        z := 0.0
        for i := 0; i < 1000; i++ {
                z -= (z*z - x) / (2 * x)
        }
        return z
}
{{< /highlight >}}

c). 编译 newmath/sqrt.go 包

{{< highlight bash >}}
1. 在任意目录下：
    $ go install example/newmath

2. 在源码目录下：
    cd $GOPATH/src/example/newmath
    $ go install

{{< /highlight >}}

d). 编译独立的可执行的测试

{{< highlight bash >}}
$ cd $GOPATH/src/example
$ mkdir hello

// $GOPATH/src/example/hello/hello.go 源码：
package main

import (
        "example/newmath"
        "fmt"
)

func main() {
        fmt.Printf("Hello, world.  Sqrt(2) = %v\n", newmath.Sqrt(2))
}

$ go install example/hello # 编译 hello

{{< /highlight >}}

这个时候可以发现在 $GOPATH/bin/hello 多了一个可执行的 hello 文件：

{{< highlight bash >}}
$ hello
Hello, world.  Sqrt(2) = 1.414213562373095
{{< /highlight >}}

完整的目录结构如下：

{{< highlight bash >}}
bin/
    hello
pkg/
    平台名/ 如：darwin_amd64、linux_amd64
        example/
            newmath.a
src/
    example/
        hello/
            hello.go
        newmath/
            sqrt.go

{{< /highlight >}}


简单的示例到此为止，官网文档中还有 go install 远程扩展库和测试相关，有兴趣的自己看把~~


