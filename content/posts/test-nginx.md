---
date: 2013-12-04
title: Test::Nginx 模块介绍
---

先说句题外话，Perl的测试模块那真是相当的爽，不仅可以爽到无与伦比的正则，还可以对测试用例自由组合、乱序运行等等。

Perl测试模块大概有如下：

* Test::Simple
* Test::More
* Test::Base 及衍生（这个我还没搞透）

一般Perl的测试用例，会在一个叫t/的目录下，一堆叫*.t的文件，其实就是普通的Perl脚本。

来段最简单的测试代码：

{{< highlight bash >}}
$ cat 1.t
use Test::Simple tests=>2;
ok(1 + 1 == 2, "test1 true");
ok("10" == 10, "test2 true");
    
$ perl 1.t
1..2
ok 1 - test1 true
ok 2 - test2 true
{{< /highlight >}}
    
Test::Simple模块只有一个ok()函数，需要复杂的一些的话就用Test::More吧。

跑偏题太远了，回到本文要介绍的Test::Nginx模块。个人觉得这个测试适用于以下场景：

* Nginx测试，包括：自身、模块、配置等测试
* 反向代理测试，比如后端的HTTP或者其他服务

安装可以用cpan或者直接clone一份Test::Nginx，库地址：[https://github.com/agentzh/test-nginx](https://github.com/agentzh/test-nginx)

文档：

* [http://search.cpan.org/~agent/Test-Nginx-0.22/lib/Test/Nginx.pm](http://search.cpan.org/~agent/Test-Nginx-0.22/lib/Test/Nginx.pm)

摘抄模块中的一段测试用例：

{{< highlight perl >}}
use Test::Nginx::Socket;
repeat_each(1);
plan tests => 2 * repeat_each() * blocks();
$ENV{TEST_NGINX_MEMCACHED_PORT} ||= 11211;  # make this env take a default value
run_tests();

__DATA__

=== TEST 1: sanity
--- config
location /foo {
    set $memc_cmd set;
    set $memc_key foo;
    set $memc_value bar;
    memc_pass 127.0.0.1:$TEST_NGINX_MEMCACHED_PORT;
}
--- request
GET /foo
--- response_body_like: STORED
--- error_code: 201
{{< /highlight >}}

可以看出就是一个基础的Perl代码，外加一部分Nginx的配置。TEST部分扩展自Test::Base：

* config，一段标准的location
* request，发起HTTP请求
* response_body_like，类似于匹配，这里可以用帅气的正则（[例子](https://github.com/smallfish/lua-resty-beanstalkd/blob/master/t/basic.t)）都可以

可以参考 [@agentzh](https://github.com/agentzh/) 很多跟Nginx有关的模块，都包含有大量的测试代码。

像我一样懒惰的人，直接copy一份，补一些业务代码就可以啦。

如果有一堆*.t测试脚本，还用perl *.t方式就不合适，Perl自带了一个工具：prove，常用参数：

{{< highlight bash >}}
-l 类似-Ilib路径，加载一些库。比如上面的Test::Nginx模块不在默认LIB下
-s 随机跑测试用例
{{< /highlight >}}

prove命令默认会找当前目录下t/文件夹下所有*.t脚本。




