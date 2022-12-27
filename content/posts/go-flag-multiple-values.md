---
date: 2015-01-17
title: Go flag/kingpin 命令行解析多个同名参数

tags:
- Golang
- Go

---

想了半天标题应该怎么写，都不太好表达，要的效果如下：

{{< highlight bash >}}
xx --name=aa --name=bb
{{< /highlight >}}

需要解析出 `name` 参数，默认 `flag` 解析后返回的是最后一个值，即：`bb`

放出 `Google` 大法，搜出一篇早期文字：[Issue 842041: Flags: add user-defined flag types](https://codereview.appspot.com/842041)

代码示例已经久远，直接拷贝代码运行会报错，原因是 `Set()` 应该返回 `error` 类型，而不是 `bool` 啦。

需要注意的是，如果想实现解析数组，需要：

* 定义类型 `type xx`
* 类型 `xx` 实现 `String()` 和 `Set()` 函数

示例输出：

{{< highlight bash >}}
$ ./xx --name=xx -name=yy
v = [xx yy]
{{< /highlight >}}

上面的示例是基于 Go 语言自带的 `flag` 模块，这个稍繁琐了点，我自己用了另外一个模块：[alecthomas/kingpin](https://github.com/alecthomas/kingpin)

作者在源码 [values.go#L203](https://github.com/alecthomas/kingpin/blob/master/values.go#L203)里也有类似的封装，但是只有一个：`Strings`（注意末尾多了一个 `s`）

同样也是实现了上述函数，另外还有一个：`IsCumulative()`

话不多说，还是上个示例：

{{< highlight go >}}
package main

import (
        "fmt"
        "gopkg.in/alecthomas/kingpin.v1"
)

var (
        names = kingpin.Flag("name", "").Strings()
)

func main() {
        kingpin.Parse()
        fmt.Println(*names)
}
{{< /highlight >}}

效果同上。



