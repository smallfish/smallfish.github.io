---
date: 2009-11-10
title: Google App Engine 上试用 web.py 笔记
---

看到有人在坛子里询问在GAE如何发布web.py有关问题，就尝试了一把。具体安装和使用过程如下，请对照自己本地路径相应修改：

1. 复制本地对应web.py目录到GAE对应应用目录

比如：D:\Python25\Lib\site-packages\web 到 e:\googleapp\pynotes

2. 写测试代码
app.yaml

{{< highlight python >}}
application: pynotes
version: 1
runtime: python
api_version: 1

handlers:
-  url: /.*
   script: home.py
{{< /highlight >}}

home.py

{{< highlight python >}}
import web
render = web.template.render('templates/')
urls = (
    '/', 'index'
)
class index:
    def GET(self):
        web.header('Content-type', 'text/html')
        name = 'smallfish'
        return render.index(name)
app = web.application(urls, globals())
main = app.cgirun() # 这行是发布到GAE的关键
{{< /highlight >}}

templates/index.html

{{< highlight python >}}
$def with (name)
hello, $name. test by web.py
{{< /highlight >}}

3. 发布到GAE，测试

e:\googleapp>appcfg.py update pynotes/

到这里，一个简单web.py应用就完成了，然后刷新。GAE显示500 Error！

看后台GAE Log显示错误信息：”No module named templates“，去web.py官方溜达了一圈，发现在其cookbook里有一篇文档《[http://webpy.org/cookbook/templates_on_gae](http://webpy.org/cookbook/templates_on_gae)》，里面说的很明白啦。

因为web.py的模板在GAE上文件系统会有所限制，所有本地得compile一下，具体命令是：python web/template.py --compile templates 最后一个参数是本地对应模板目录templates，如果有多个模板目录则一次运行一次。运行完会在templates会生成一个__init__.py，里面内容有兴趣可以看看，很眼熟的哦。

4. 再次发布到GAE，可以看到OK拉！

