---
date: 2010-05-30
title: Go 语言模块安装工具：goinstall
---

<img src="http://golang.org/doc/logo-153x55.png" alt="" />

文档地址：[http://golang.org/cmd/goinstall/](http://golang.org/cmd/goinstall/)

goinstall 主要是方便安装第三方模块，目前支持 hg(mercurial)，git，svn三种版本控制系统。

下面来举例怎么安装 web.go 模块。源地址是：[http://github.com/hoisie/web.go](http://github.com/hoisie/web.go)

{{< highlight python >}}
smallfish@debian:~$ goinstall -dashboard=true github.com/hoisie/web.go
{{< /highlight >}}

根据网速快慢，过一段时间会结束。期间木有任何提示。（可以加上 -v=true 参数，可以显示安装过程和提示。）

查看下安装的目录和路径：

{{< highlight python >}}
smallfish@debian:~$ ls $GOROOT/src/pkg/github.com/hoisie/web.go/
examples  _go_.8   Makefile  Readme.md   scgi.go       status.go  web_test.go
fcgi.go   LICENSE  _obj      request.go  servefile.go  web.go
{{< /highlight >}}

代码示例：

{{< highlight python >}}
import (web "github.com/hoisie/web.go")
{{< /highlight >}}

另外注意点，官方文档里 -update 选项现在版本里已经缩写，改成 -u。

