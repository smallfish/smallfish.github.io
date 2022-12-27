---
date: 2009-12-15
title: Apache Mod_Perl实现 URL Rewrite
---

相信apache的mod_rewrite模块都很熟悉了，今天这儿换个思路，利用mod_perl来实现下，发现竟然是如此的简单！

首先得保证apache已经安装了mod_perl模块，具体安装配置可以看上一篇文章哦。

修改下http.conf配置，添加一下内容：

{{< highlight python >}}

PerlTransHandler MyTrans # MyTrans 这个是自己添加的处理模块名

{{< /highlight >}}

具体MyTrans.pm脚本如下：

{{< highlight python >}}

package MyTrans;

use strict;
use Apache2::Const qw(DECLINED);

sub handler {
    my $r = shift;
    my $uri = $r->uri;
    my ($id) = ($url =~ m|^/news/(.*)\.html|);
    $r->uri("/news.php");
    $r->args("id=$id");
    return Apache2::Const::DECLINED;
}
1;

{{< /highlight >}}

实现就是：/news/12345.html  => /news.php?id=12345

