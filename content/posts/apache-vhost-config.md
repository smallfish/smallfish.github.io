---
date: 2009-12-15
title: Apache 虚拟主机配置笔记

tags:
- Apache

---

环境：Linux Apache2.2 （路径 /usr/local/apache）

步骤：

1. 修改 conf/httpd.conf，找到如下位置，去除 # 注释符

{{< highlight python >}}

# Virtual hosts
Include conf/extra/httpd-vhosts.conf

{{< /highlight >}}

2.修改 conf/extra/httpd-vhosts.conf

{{< highlight python >}}

<VirtualHost *:80>
    ServerAdmin webmaster@aa.com
    DocumentRoot "/usr/aa"
    ServerName ww.aa.com
    ServerAlias ww.aa.com
    ErrorLog "logs/ww.aa.com-error_log"
    CustomLog "logs/ww.aa.com-access_log" common
</VirtualHost>

{{< /highlight >}}

注意CustomLog这行，默认给的配置是："logs/dummy-host.example.com-access_log common"

这个其实是错误的，Apache启动时候会报错，common这个应该放在双引号外面：

{{< highlight python >}}

Syntax error on line 32 of /usr/local/apache/conf/extra/httpd-vhosts.conf:
CustomLog takes two or three arguments, a file name, a custom log format string or format name, and an optional "env=" clause (see docs)

{{< /highlight >}}

另外还有个问题，在Apache+Tomcat时候出现的，在配置好mod_jk之后，通过默认80端口访问jsp总会提示403禁止访问，纳闷了！后来 才发现是Directory配置问题，因为每个VirtualHost配置的目录不是默认的DocumentRoot目录，在Apache2以后对于权限 和安全有了更高的要求，所以必须手动配置下每个Directory属性。

{{< highlight python >}}

<Directory "/usr/aa">
    Options Indexes FollowSymLinks
    AllowOverride None
    Order allow,deny
    Allow from all
</Directory>

{{< /highlight >}}

OK，到这里虚拟主机部分已经配置完成！

补充一点，加上日志按天保存，避免一个巨大无比的文件！

{{< highlight python >}}

CustomLog "|/usr/local/apache/bin/rotatelogs /usr/local/apache/logs/ww.aa.com-access-%y%m%d.log 86400 480 " combined

{{< /highlight >}}


