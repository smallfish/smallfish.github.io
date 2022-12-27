---
date: 2010-03-27
title: Nginx 启动/重启脚本笔记
---

Nginx本身可以通过

{{< highlight python >}}
kill -HUP `cat /usr/local/nginx/logs/nginx.pid`

{{< /highlight >}}

进行平滑重启的，可以通过ps进程查看一下。效果还是挺不错的。

这里介绍的是另外一种方式service，适合RHEL/CentOS系列。

1. kill nginx进程

{{< highlight python >}}
kill `cat /usr/local/nginx/logs/nginx.pid`

{{< /highlight >}}

2. 建立 /etc/init.d/nginx 文件

{{< highlight python >}}
#!/bin/sh
#
# nginx - this script starts and stops the nginx daemin
#
# chkconfig:   - 85 15
# description:  Nginx is an HTTP(S) server, HTTP(S) reverse \
#               proxy and IMAP/POP3 proxy server
# processname: nginx
# config:      /usr/local/nginx/conf/nginx.conf
# pidfile:     /usr/local/nginx/logs/nginx.pid

# Source function library.
. /etc/rc.d/init.d/functions

# Source networking configuration.
. /etc/sysconfig/network

# Check that networking is up.
[ "$NETWORKING" = "no" ] &amp;&amp; exit 0

nginx="/usr/local/nginx/sbin/nginx"
prog=$(basename $nginx)

NGINX_CONF_FILE="/usr/local/nginx/conf/nginx.conf"

lockfile=/var/lock/subsys/nginx

start() {
    [ -x $nginx ] || exit 5
    [ -f $NGINX_CONF_FILE ] || exit 6
    echo -n $"Starting $prog: "
    daemon $nginx -c $NGINX_CONF_FILE
    retval=$?
    echo
    [ $retval -eq 0 ] &amp;&amp; touch $lockfile
    return $retval
}

stop() {
    echo -n $"Stopping $prog: "
    killproc $prog -QUIT
    retval=$?
    echo
    [ $retval -eq 0 ] &amp;&amp; rm -f $lockfile
    return $retval
}

restart() {
    configtest || return $?
    stop
    start
}

reload() {
    configtest || return $?
    echo -n $"Reloading $prog: "
    killproc $nginx -HUP
    RETVAL=$?
    echo
}

force_reload() {
    restart
}

configtest() {
  $nginx -t -c $NGINX_CONF_FILE
}

rh_status() {
    status $prog
}

rh_status_q() {
    rh_status >/dev/null 2>&amp;1
}

case "$1" in
    start)
        rh_status_q &amp;&amp; exit 0
        $1
        ;;
    stop)
        rh_status_q || exit 0
        $1
        ;;
    restart|configtest)
        $1
        ;;
    reload)
        rh_status_q || exit 7
        $1
        ;;
    force-reload)
        force_reload
        ;;
    status)
        rh_status
        ;;
    condrestart|try-restart)
        rh_status_q || exit 0
            ;;
    *)
        echo $"Usage: $0 {start|stop|status|restart|condrestart|try-restart|reload|force-reload|configtest}"
        exit 2
esac
{{< /highlight >}}

3. 设置脚本为可执行

{{< highlight python >}}
chmod +x /etc/init.d/nginx

{{< /highlight >}}

4.

{{< highlight python >}}
/sbin/chkconfig nginx on

{{< /highlight >}}

5. 检查下

{{< highlight python >}}
/sbin/chkconfig --list nginx
nginx           0:off   1:off   2:on    3:on    4:on    5:on    6:off

{{< /highlight >}}

完成咯~可以尝试下面方式启动或者重启

{{< highlight python >}}
service nginx start
service nginx stop
service nginx restart
service nginx reload

/etc/init.d/nginx start
/etc/init.d/nginx stop
/etc/init.d/nginx restart
/etc/init.d/nginx reload

{{< /highlight >}}


