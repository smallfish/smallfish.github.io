---
date: 2011-02-19
title: Pylons 入门实例教程 – 发布应用

tags:
- Python

---

前面几篇教程简单讲述了如何使用 Pylons 进行 WEB 方面开发的步骤，包括简单的 Hello、表单和数据库操作等内容。本篇将描述一下如何在正式环境中发布基于 Pylons 的 WEB 应用。

测试环境：Nginx 0.8.53 + FastCGI 模式 （需要安装 flup 模块）

{{< highlight python >}}
pip install flup
# easy_install -U flup
{{< /highlight >}}

测试代码，延用前面的 Hello 示例。

{{< highlight python >}}
mac:python smallfish$ paster create -t pylons hello
mac:python smallfish$ cd hello/
mac:hello smallfish$ paster controller hi
mac:hello smallfish$ paster serve --reload development.ini
{{< /highlight >}}

确保以上过程无错，访问：http://127.0.0.1:5000 可以看到默认页面。

好把，开始配置发布环境。
需要修改 development.ini 配置文件，找到 [server:main] 节点，修改其中的 use 方式（默认是 egg:Paste#http）。

{{< highlight python >}}
[server:main]
use = egg:PasteScript#flup_fcgi_thread
host = 0.0.0.0
port = 5000
{{< /highlight >}}

另外建议修改 [DEFAULT] 节点中 debug = false，以免错误会打印具体的环境和堆栈信息。

到这里 Pylons 部分以及设置好了，如果出现一下错误：

{{< highlight python >}}
LookupError: Entry point 'flup_fcgi_thread' not found in egg 'Paste'
{{< /highlight >}}

请注意上面的 use 部分里是 PasteScript 而不是 Paste哦。淡定淡定，别光看后面的 http 改成 flup_fcgi_thread。

下面修改 Nginx 的配置 nginx.conf，修改其中 / 部分设置。

{{< highlight python >}}
        location / {
            fastcgi_pass   127.0.0.1:5000;
            fastcgi_index  index;
            fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
            include        fastcgi_params;
            fastcgi_intercept_errors off;
        }
{{< /highlight >}}

其中 fastcgi_pass 中 host 和 port 对应 development.ini 中的配置。

好把，这里可以启动，看看是否正确，请注意相应的目录。

启动 Pylons 部分（启动 daemon 进程，或参考搜狐邮箱使用的 [http://mailteam.blog.sohu.com/136961251.html](http://mailteam.blog.sohu.com/136961251.html) 来管理进程）：

{{< highlight python >}}
mac:hello smallfish$ paster serve --reload development.ini  --daemon start
# 注意，正式环境中请去除 reload 参数
# daemon 后面参数有 start | stop | restart | status，可以很方便的管理进程
{{< /highlight >}}

启动 Nginx 部分：

{{< highlight python >}}
mac:~ smallfish$ sudo /opt/nginx/sbin/nginx
{{< /highlight >}}

尝试一下访问： http://localhost/ 和 http://localhost/hi/index 。是不是都可以了？OK，到这里已经整合好了。

这里是采用 FastCGI 方式发布，当然还可以用默认 http 或者本地 socket 方式，然后通过 proxy_pass 方式转发过去也是可以的。

当然，这个只是简单示例，可能正式发布时候需要注意的事项比这个复杂的多。这里只是介绍一种方式而已。

希望通过这一系列的教程，希望能对喜欢 Python 和关注 Pylons 框架的朋友能有所帮助。

前几篇链接分别如下：

1. [Pylons 入门实例教程 - Hello](http://chenxiaoyu.org/2010/06/28/pylons-tutorial-hello.html)
2. [Pylons 入门实例教程 – 表单和文件上传](http://chenxiaoyu.org/2010/06/30/pylons-tutorial-form-upload-file.html)
3. [Pylons 入门实例教程 – 数据库操作](http://chenxiaoyu.org/2010/07/01/pylons-tutorial-database.html)
4. [Pylons 入门实例教程 – cookie 和 session](http://chenxiaoyu.org/2010/07/03/pylons-tutorial-cookie-session.html)

