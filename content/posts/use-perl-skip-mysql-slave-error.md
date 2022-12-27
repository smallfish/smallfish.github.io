---
date: 2009-12-15
title: Perl 批量跳过 MySQL Slave 复制错误
---

发现mysql slave服务器经常因为一些特殊字符或者符号产生的更新语句报错，整个同步也会因此而卡在那，最初的办法只是手动去出错的机器，执行下面三条sql语句，跳过错误即可。

{{< highlight python >}}

slave stop;
set GLOBAL SQL_SLAVE_SKIP_COUNTER=1;
slave start;

{{< /highlight >}}

一台slave机器用这样方法还行，多台就麻烦了，就顺手写了个简单的perl脚本，方便统一跳过错误，代码如下：

{{< highlight python >}}

#!/usr/bin/env perl
use strict;
use warnings;

# get slave status
sub get_status {
    my ($ip, $usr, $pass) = @_;
    my $info = `mysql -u$usr -p$pass -h$ip -e 'show slave status\\G;'`;
    if (($info =~ /Slave_IO_Running: Yes/) &amp;&amp; ($info =~ /Slave_SQL_Running: No/)) {
        return 1;
    }
    return 0;
}
# mysql slave skip
sub slaveskip {
    my ($ip, $usr, $pass) = @_;
    print "slave error **\n";
    system("mysql -u$usr -p$pass -h$ip -e 'slave stop;'");
    system("mysql -u$usr -p$pass -h$ip -e 'set GLOBAL SQL_SLAVE_SKIP_COUNTER=1;'");
    system("mysql -u$usr -p$pass -h$ip -e 'slave start;'");
}

my @hosts = qw/
192.168.0.101:root:tt1234
192.168.0.102:root: tt1234
192.168.0.103:root: tt1234
/;
foreach (@hosts) {
    my ($ip, $usr, $pass) = split ':';
    print "// ----- $ip\n";
    my $count = 1;
    while ($count < 100000) {
        my $status = get_status($ip, $usr, $pass);
        print "i: $count status: $status\n";
        last if $status == 0;
        slaveskip($ip, $usr, $pass);
        select(undef, undef, undef, 0.1);
        $count++;
    }
    print "\n";
}

exit;

{{< /highlight >}}


