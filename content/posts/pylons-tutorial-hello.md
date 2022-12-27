---
date: 2010-06-28
title: Pylons 入门实例教程 - Hello
---

[http://pylonshq.com/](http://pylonshq.com/)，当然，这些组件只是默认，你还可以根据自己喜好来选择其他组件，比如你可以采用 Jinja2 或 Genshi 模板，ORM也可以采用 SQLObject。完全是自由组合。

废话少说，现在开始安装吧。

{{< highlight python >}}
smallfish@debian:~$ sudo aptitude install python-pylons
{{< /highlight >}}

Debian/Ubuntu 系列系统可以直接 aptitude 安装，当然你也可以使用 easy_install 或者源码安装。

{{< highlight python >}}
smallfish@debian:~$ sudo easy_install Pylons
{{< /highlight >}}

更多安装文档请参考官网安装部分，[http://pylonshq.com/docs/en/1.0/gettingstarted/#installing](http://pylonshq.com/docs/en/1.0/gettingstarted/#installing)

好了，安装结束，来一个经典的Hello程序吧。

{{< highlight python >}}
smallfish@debian:~/workspace/python$ paster create -t pylons hello
Selected and implied templates:
  Pylons#pylons  Pylons application template

Variables:
  egg:      hello
  package:  hello
  project:  hello
Enter template_engine (mako/genshi/jinja2/etc: Template language) ['mako']:
Enter sqlalchemy (True/False: Include SQLAlchemy 0.5 configuration) [False]:
Creating template pylons
Creating directory ./hello
{{< /highlight >}}

下面输出略过，大致解说一下。Pylons 程序可以用 Paste 自动生成一些代码，包括controller。还可以运行 HTTP 服务来测试。

-t  表示自动创建的模板，可以如下来查看有哪些选项，更多就参考 help 吧。

{{< highlight python >}}
smallfish@debian:~/workspace/python/hello$ paster create --list-templates
Available templates:
  basic_package:   A basic setuptools-enabled package
  paste_deploy:    A web application deployed through paste.deploy
  pylons:          Pylons application template
  pylons_minimal:  Pylons minimal application template
{{< /highlight >}}

看一下hello的目录结构：

{{< highlight python >}}
smallfish@debian:~/workspace/python/hello$ ls
development.ini  ez_setup.py  hello.egg-info  README.txt  setup.py
docs             hello        MANIFEST.in     setup.cfg   test.ini
{{< /highlight >}}

这里具体各个文件意思就不讲解了，程序主体部分都在hello/hello目录下，development.ini 和 test.ini 分别是服务启动的配置文件，用于测试和开发环境。开始先运行一下，看效果吧。。

{{< highlight python >}}
smallfish@debian:~/workspace/python/hello$ paster serve --reload development.ini
Starting subprocess with file monitor
Starting server in PID 1519.
serving on http://127.0.0.1:5000
{{< /highlight >}}

在浏览器中打开：http://127.0.0.1:5000 看到页面了吧？恭喜。

继续，写一个简单的显示 hi的 controller 程序吧。

{{< highlight python >}}
smallfish@debian:~/workspace/python/hello$ paster controller hi
Creating /home/smallfish/workspace/python/hello/hello/controllers/hi.py
Creating /home/smallfish/workspace/python/hello/hello/tests/functional/test_hi.py
{{< /highlight >}}

自动生成程序和 test 文件。paster 启动服务不需要重启会自动加载，可以浏览器访问：http://127.0.0.1:5000/hi/index

很简单吧，打开 hi.py，基本如下：

{{< highlight python >}}
class HiController(BaseController):

    def index(self):
        # Return a rendered template
        #return render('/hi.mako')
        # or, return a response
        return 'Hello World'
{{< /highlight >}}

上面注释部分可以 return 一个模板文件，模板放入 templates 目录下即可。

去除上面的 return 'Hello' 返回 return render('/hi.mako')

{{< highlight python >}}
smallfish@debian:~/workspace/python/hello$ vi hello/templates/hi.mako
% for key, value in request.environ.items():
 ${key} = ${value}
% endfor
{{< /highlight >}}

刷新 http://127.0.0.1:5000/hi/index ，可以看到一些环境变量的输出了吧。

今天就简单的说到这里吧，下回来一个完整的例子，包括URL、模板、数据库和Session的实例。

