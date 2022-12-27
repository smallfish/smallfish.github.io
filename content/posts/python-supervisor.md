---
date: 2011-05-31
title:  supervisor - Python进程管理工具
---

经常会碰到要写一些守护进程，简单做法放入后台：

{{< highlight python >}}
shell> nohup python xxx.py &
{{< /highlight >}}

偶尔这么做还可以接受，如果一堆这样的呢？

当然还有一个问题，就是各种服务，对应的命令或者路径都不太一致，比如Apache、MySQL或者其他自行编译的工具。

如果可以统一管理这些应用，是不是很哈皮？

按照惯例现Google一把，不失所望找到一个神奇的利器。supervisor！

supervisor地址：[http://supervisord.org](http://supervisord.org)，官方标语就是：一个进程管理工具。

安装：

{{< highlight python >}}
shell> sudo aptitude install supervisor # pip/easy_install
{{< /highlight >}}

也可以通过其他包管理来安装，比如apt/yum等。

安装好以后，有两个可执行文件和一个配置文件（平台差异，可能路径不一致）：

{{< highlight python >}}
/usr/bin/supervisord             --  supervisor服务守护进程
/usr/bin/supervisorctl           --  supervisor服务控制程序，比如：status/start/stop/restart xx 等
/etc/supervisor/supervisord.conf --  配置文件，定义服务名称以及接口等等
{{< /highlight >}}

下面来一个示例，用web.py写一个hello的程序：

{{< highlight python >}}

import web

urls = (
    '/(.*)','hello'
)

app = web.application(urls, globals())

class hello:
    def GET(self, name):
        return 'hello: ' + name

if __name__ == '__main__':
    app.run() 

{{< /highlight >}}

这个时候可以直接启动这个程序了，下面来配置supervisor，加入管理。修改supervisord.conf，加入如下片段：

{{< highlight html >}}
[program:hello]
command=python /home/smallfish/hello.py
autorstart=true
stdout_logfile=/home/smallfish/hello.log
{{< /highlight >}}

上面的意思应该很容易懂，program后面跟服务的名称，command是程序的执行路径，autorstart是表示自动启动，stdout_logfile是捕获标准输出。

到这里，基本搞定了，下面就是启动管理：

{{< highlight python >}}
shell> sudo /etc/init.d/supervisor start   -- 启动supervisor服务

shell> sudo supervisorctl status hello     -- 获取hello服务的状态，因为是autorstart，这里已经启动了
hello  RUNNING    pid 1159, uptime 0:20:32

shell> sudo supervisorctl stop hello       -- 停止hello服务
hello: stopped

shell> sudo supervisorctl stop hello       -- 再次停止hello，会有错误信息
hello: ERROR (not running)

shell> sudo supervisorctl start hello      -- 启动hello服务
hello: started

{{< /highlight >}}

OK，基本的操作就是类似这个了，仔细看supervisord.conf文件里会发现有一段[unix_http_server]的配置，默认是9001端口，可以输入用户名和密码，主要用于Basic Auth认证用的。

填写一下，然后重启supervisor服务，打开浏览器输入：http://localhost:9001，如图：

![](/images/supervisor-web.png)
