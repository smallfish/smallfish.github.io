---
date: 2010-06-30
title: Pylons 入门实例教程 – 表单和文件上传
---

继续上一篇《[Pylons 入门实例教程 - Hello](http://chenxiaoyu.org/2010/06/28/pylons-tutorial-hello.html)》，现在开始讲在 Pylons 里如何提交表单和上传文件。

继续延用上篇里面的 hello 工程，在 HiController 里添加 form 方法：

{{< highlight python >}}
    def form(self):
        return render('/form.mako')
{{< /highlight >}}

加完以后可以访问：http://127.0.0.1:5000/hi/form，会报错。

Server Error，根据报错内容大致就知道模板文件不存在了。如果有其他错误，也可以通过这个页面查看，当然还有很强大的 Debug 个功能哦。当然正式环境一般都是关闭这个功能的。这个，你懂得。。。

好吧，写一个表单的模板，只包含一个简单的文本框和提交按钮示例。

{{< highlight python >}}
<form action="/hi/submit" method="post">
name: <input type="text" name="name" />
<br />
<input type="submit" value="submit" />
</form>
{{< /highlight >}}

再添加一个 submit 方法来处理表单提交，

{{< highlight python >}}
    def submit(self):
        return "hello, name: %s" % request.params['name']
{{< /highlight >}}

request.params 包含了表单或者URL提交的参数，建议 POST 数据参照下面的上传部分。想获取更详细的列表，可以查看文档或者自己手动 dir()查阅。

下面尝试一下文件上传，首先在 development.ini 添加一个变量，用来存放文件上传后的文件夹。

{{< highlight python >}}
[app:main]
upload_dir = %(here)s/upload
{{< /highlight >}}

%(here) 启动后 server 会替换到当前目录地址，上面的地址就是当前路径下的upload文件夹。

修改一下刚才的表单，加一个 file 上传，注意 multipart/form-data 这句，上传必须。

{{< highlight python >}}
<form action="/hi/submit" method="post"  enctype= "multipart/form-data">
name: <input type="text" name="name" />
<br />
file: <input type="file" name="file" />
<br />
<input type="submit" value="submit" />
</form>
{{< /highlight >}}

修改 submit 方法，添加文件内容：

{{< highlight python >}}
    def submit(self):
        name   = request.POST['name']
        myfile = request.POST['file']
        import os
        import shutil
        from pylons import config
        local_name = os.path.join(config['app_conf']['upload_dir'], myfile.filename)
        local_file = open(local_name, "wb")
        shutil.copyfileobj(myfile.file, local_file)
        myfile.file.close()
        local_file.close()
        return "hello, name: %s, upload: %s" % (name, myfile.filename)
{{< /highlight >}}

里面 import 部分这里仅仅为了示例，正式的代码请放入程序开头部分，POST 内容可以从 request.POST 获取。

config['app_conf']['upload_dir'] 就是刚才配置里 development.ini 定义的地址。这个目录需要自己手动创建哦。

{{< highlight python >}}
smallfish@debian:~/workspace/python/hello$ mkdir upload
{{< /highlight >}}

OK，到这里程序部分都已经修改完成。重新访问一下：http://127.0.0.1:5000/hi/form

尝试一下上传，上传后可以在 upload 文件夹下看到文件了吧。。

当然这里只是示例，还需要处理一下上传的名字，防止有特殊符号哦。

