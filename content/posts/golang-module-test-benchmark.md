---
date: 2012-12-07
title: Go 模块测试

tags:
- Golang
- Go

---

Go 很多地方都透露着“约定大于配置”的理论，比如测试、可见性、语法等等。

本文示例模块为：foo.go，则对应的测试模块为：foo_test.go，测试版本为：go v1.0.3。

先写好示例代码： foo.go

{{< highlight bash >}}
package foo

func Add(a, b int) int {
    return a + b
}
{{< /highlight >}}

对应的测试代码：foo_test.go

{{< highlight bash >}}
package foo

import "testing"

func TestAdd(t *testing.T) {
    if (Add(1, 2) != 3) {
        t.Error("test foo:Addr failed")
    } else {
        t.Log("test foo:Addr pass")
    }
}   
{{< /highlight >}}

到这里可以运行测试了：

{{< highlight bash >}}
$ go test
PASS
ok      _/Users/smallfish/test/go/foo   0.080s
{{< /highlight >}}
   
或者详细一点的输出：

{{< highlight bash >}}
$ go test -v
=== RUN TestAdd
--- PASS: TestAdd (0.00 seconds)
foo_test.go:9:  test foo:Addr pass
                PASS
ok      _/Users/smallfish/test/go/foo   0.013s
{{< /highlight >}}

默认测试函数是以“Test”开头，比如：TestXXX。而性能测试代码是以“Benchmark”开头。

下面来简单一段性能测试代码：

{{< highlight bash >}}
func BenchmarkAdd(b *testing.B) {
    // 如果需要初始化，比较耗时的操作可以这样：
    // b.StopTimer()
    // .... 一堆操作
    // b.StartTimer()
    for i := 0; i < b.N; i++ {
        Add(1, 2)
    }
}
{{< /highlight >}}

跑一下性能测试：

{{< highlight bash >}}
$ go test -test.bench=".*"
PASS
BenchmarkAdd    2000000000               1.27 ns/op
ok      _/Users/smallfish/test/go/foo   2.702s
{{< /highlight >}}

更多请参考：“[go](http://golang.org/cmd/go/) help test” 命令和 [testing](http://golang.org/pkg/testing/) 模块


