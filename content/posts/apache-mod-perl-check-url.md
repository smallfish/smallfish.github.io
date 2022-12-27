---
date: 2009-12-15
title: Apache Mod_Perl 防盗链

tags:
- Apache
- Perl

---

大体思路是这样的，比如有 一个地址：http://www.aa.com/down/1.mp3，不幸搜索引擎或者迅雷扒到了，就无偿为他们奉献流量了。 但是假如在http://www.aa.com/down/1.mp3?key=123，key参数每天变化或者几分钟变化一次，在apache服务端校 验下这个参数，不正确则显示拒绝访问或者找不到的话，那样防盗链的效果就算达到了把。

modperl强大到可以任意应用apache内部API，官方地址是：[http://perl.apache.org/](http://perl.apache.org/) 。根据apache版本选择相应的modperl版本，现在大部分都apache2了，就选择modperl2把。具体安装配置可以看官方文档。

先建立/home/httpd/modperl/startup.pl(目录请自行修改)，内容如下：

{{< highlight python >}}
use strict;

use lib qw(/home/httpd/modperl); # 把这个路径加入到perl lib路径

use Apache2::RequestRec ();
use Apache2::RequestIO ();
use Apache2::Connection ();
use Apache2::RequestUtil ();
use Apache2::ServerUtil ();
use Apache2::Log ();
use Apache2::Request ();

1;
{{< /highlight >}}

部分本机httpd.conf配置：

{{< highlight python >}}

LoadModule perl_module                modules/mod_perl.so
LoadModule apreq_module             modules/mod_apreq2.so
PerlPostConfigRequire /home/httpd/modperl/startup.pl
<Location /down >
    SetHandler modperl # 设置该目录交给modper处理
    PerlAccessHandler Down # Down是模块名称
    PerlSetVar key 123 # 设置校验参数值
</Location>

{{< /highlight >}}

mod_apreq2.so这个模块需要安装Apache2::Request，具体安装：[http://pyh7.spaces.live.com/blog/cns%2147D8D44208AC51E5%21128.entry](http://pyh7.spaces.live.com/blog/cns%2147D8D44208AC51E5%21128.entry)

startup.pl文件一般modperl应用都有，加载一些常用库，可以在apache启动时预先载入，避免重复加载。

修改这些后可以重启下apache，看下logs/error_log里最后是否有mod_apreq和mod_perl字样，如果有就说明成功了。剩下都就是写校验的perl脚本了，/home/httpd/modperl/Down.pm，内容如下：

{{< highlight python >}}

package Down;
use strict;
use Apache2::RequestRec ();
use Apache2::RequestIO ();
use Apache2::Connection ();
use Apache2::RequestUtil ();
use Apache2::ServerUtil ();
use Apache2::Log ();
use Apache2::Const -compile => qw(OK FORBIDDEN);
use Apache2::Request ();

sub handler {
    my $r = shift;
    my $req = Apache2::Request->new($r);
    my $ip = $r->connection->remote_ip;
    my $k = $req->param('key') || ''; # 判断访问时是否带key参数
    my $key = $r->dir_config('key') || '123'; # 加载httpd.conf配置中的key值
    if ($key eq $k) { # 相等可以正常访问
        return Apache2::Const::OK;
    } else { # 否则显示拒绝访问
        my $s = Apache2::ServerUtil->server;
        $s->log_error("[$ip--------forbidden.]");
        return Apache2::Const::FORBIDDEN;
    }
}

1;

{{< /highlight >}}

提示一下，以上两个perl脚本文件末尾记得加上1;

重启下apache。现在可以通过http://www.aa.com/down/1.mp3和http://www.aa.com/down /1.mp3?key=123测试下。可以看见不加参数拒绝访问了把。这里只是简单的判断，实际上可以根据日期或者IP加密生成一个字符串来判断。

写这个帖子完全是无意中搜索modperl应用时候发现了，具体可以参见：
[http://pyh7.spaces.live.com/blog/cns%2147D8D44208AC51E5%21140.entry](http://pyh7.spaces.live.com/blog/cns%2147D8D44208AC51E5%21140.entry)

上面的文档已经写都很详细了，包括怎么安装modperl、Apache2::Request等模块以及配置apache的http.conf就不在累赘都重复了。

