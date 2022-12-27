---
date: 2009-12-15
title: 使用 Perl 快速解析 Apache Log
---

功能简述

统计出日志里一个或多个页面总共访问的次数，比如aa.jsp, bb.jsp这样页面分别多少次。

实现简述

Apache单个日志文件800M。

最初程序使用Python编写，按行来统计，分别使用in(最慢)和index方法去查找，然后使用了正则匹配，程序运行时间从最初的1分50多秒优化到1分10秒左右，参考了[http://www.dup2.org/node/1006](http://www.dup2.org/node/1006)博客中提到的gc.disable()，有了一定的提升，最终还是需要1分左右。

然后随意用了Perl写了一个，用了最土鳖的<LOG>这样的按行分析，最后正则匹配，然后++，速度竟然在40-50秒之间，惊叹！后来经过[http://shucho.org/blog/](http://shucho.org/blog/)指点，在正则部分采用了预编译，效果那是相当惊人！800M文件只用了7秒左右。卧槽！

程序片段

{{< highlight python >}}
# --------------------------------------------------------------------
use strict;
use Benchmark;

my $LOG_FILE = '/usr/local/apache/logs/access.log';
# 下面qr部分起了关键作用，预编译了表达式
my @EXT_LIST = map {qr/$_/} qw{
aaServlet
bbServlet
};

my $startime = new Benchmark;
my %result;
map {$result{$_} = 0} @EXT_LIST;
open LOG_FILE, $LOG_FILE;
while (<LOG_FILE>){
    foreach my $ext (@EXT_LIST){
        $result{$ext}++ if $_ =~ /$ext/;
    }
}
close LOG_FILE;

while (my ($key, $value) = each(%result)){
    $key =~ s/\(\?-xism:(.*?)\)/$1/g;
    print "$key:\t$value\n";
}

printf "** %s\n\n", timestr(timediff(new Benchmark, $startime));
{{< /highlight >}}


