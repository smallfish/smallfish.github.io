---
date: 2009-11-10
title: Python Mako Template 学习笔记

tags:
- Python

---

Mako是什么？Moko是Python写的一个模板库，Python官网[http://python.org/](http://python.org/)用的就是它哦。其他废话也就不累赘了，直接来点代码，方便阅读与了解把。

(Mako官网地址：[http://www.makotemplates.org/](http://www.makotemplates.org/) ，可以下载安装包，推荐使用easy_install安装)

{{< highlight python >}}
from mako.template import Template
mytemplate = Template("hello world!")
print mytemplate.render()
mytemplate = Template("hello, ${name}!")
print mytemplate.render(name="jack")
{{< /highlight >}}

代码可以参考官方doc部分

{{< highlight python >}}
mytemplate = Template(filename='/docs/mytmpl.txt')
print mytemplate.render()
{{< /highlight >}}

还可以从设置模板为文件，设置filename属性

{{< highlight python >}}
mytemplate = Template(filename='/docs/mytmpl.txt', module_directory='/tmp/mako_modules')
print mytemplate.render()
{{< /highlight >}}

文件还可以缓存到某个目录下，下面的/docs/mytmpl.txt会产生一个py：/tmp/mako_modules/docs/mytmpl.txt.py

{{< highlight python >}}
from mako.lookup import TemplateLookup
mylookup = TemplateLookup(directories=['/docs'])
mytemplate = Template("""<%include file="header.txt"/> hello world!""", lookup=mylookup)
{{< /highlight >}}

查找模板，方便统一模板路径使用。

{{< highlight python >}}
mylookup = TemplateLookup(directories=['/docs'], module_directory='/tmp/mako_modules')
def serve_template(templatename, **kwargs):
mytemplate = mylookup.get_template(templatename)
print mytemplate.render(**kwargs)
{{< /highlight >}}

改良了上面的查找方式

{{< highlight python >}}

mylookup = TemplateLookup(directories=['/docs'], output_encoding='utf-8',
                                       encoding_errors='replace')
mytemplate = mylookup.get_template("foo.txt")
print mytemplate.render()
{{< /highlight >}}

设置输出编码，以及编码错误时候处理方式

