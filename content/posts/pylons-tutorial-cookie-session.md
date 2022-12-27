---
date: 2010-07-03
title: Pylons 入门实例教程 – cookie 和 session

tags:
- Python

---

本篇讲述在 Pylons 里使用 cookie 和 session。

示例还是在上篇《[Pylons 入门实例教程 – cookie 和 session](http://chenxiaoyu.org/2010/07/01/pylons-tutorial-database.html)》的代码里继续添加。先来尝试下  cookie，添加新的 cookietest controller。

修改 index 方法，添加显示部分：

{{< highlight python >}}
    def index(self):
        name = 'NULL'
        if request.cookies.has_key('name'):
            name = request.cookies['name']
        return 'cookie name=%s' % name
{{< /highlight >}}

cookie 读取可以通过 request.cookies 对象，类似一个字典结构。需要注意的是读取时候用最好 has_key 判断下，这样避免抛 KeyError 异常。当然你也可以 try...catch 捕获一下。

再重新写一个方法，用来写 cookie。

{{< highlight python >}}
    def writecookie(self):
        response.set_cookie("name", "smallfish")
        return "write cookie ok"
{{< /highlight >}}

这里只是简单设置一个值得，set_cookie 还有其他参数，具体如下：

{{< highlight python >}}
set_cookie(self, key, value='', max_age=None, path='/', domain=None, secure=None,
                 httponly=False, version=None, comment=None, expires=None, overwrite=False)
{{< /highlight >}}

基本一般需要设置：max_age，path，domain，expires 这几个参数。

下面再来尝试一下 session：

{{< highlight python >}}
smallfish@debian:~/workspace/python/hellodb$ paster controller sessiontest
Creating /home/smallfish/workspace/python/hellodb/hellodb/controllers/sessiontest.py
Creating /home/smallfish/workspace/python/hellodb/hellodb/tests/functional/test_sessiontest.py
{{< /highlight >}}

和上面 cookie 例子类似，在 controller 里有两个方法，index 负责显示 session，writesession 负责写。

{{< highlight python >}}
    def index(self):
        name = session.get('name', 'NULL')
        return 'session name=%s' % name

    def writesession(self):
        session['name'] = 'smallfish'
        session.save()
        return "save session ok"
{{< /highlight >}}

index 方法里 get 后面的 NULL 是默认值。writesession 里需要注意下设置 session 之后需要 save。

删除 session 可以尝试如下：

{{< highlight python >}}
del session['name']
# 删除所有
session.delete()
{{< /highlight >}}

到这里，WEB 常用的一些东西在 Pylons 里基本走了一圈，包含 URL、模板、数据库和会话部分。

下一节将会涉及怎么在 Nginx 上发布 Pylons 应用。

