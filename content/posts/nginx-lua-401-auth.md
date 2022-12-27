---
date: 2012-02-08
title: Nginx-Lua HTTP 401 认证校验
---

本文示例依赖模块：

* [lua-nginx-module](https://github.com/chaoslawful/lua-nginx-module/tags)
* [ngx_coolkit](https://github.com/FRiCKLE/ngx_coolkit)（获取$remote_passwd 输入值）

如何编译Nginx这些扩展模块，请参考以前[《Nginx 第三方模块试用记》](http://chenxiaoyu.org/2011/10/30/nginx-modules.html)。

基于 HTTP 401 认证其实很简单，语言或者 Web 服务器都可以在 Header 头里发送。

比如 PHP [401发送示例](http://php.net/manual/zh/features.http-auth.php)，或者 Apache 配置：

{{< highlight html >}}

<Directory "/path/a">
  AuthType Basic
  AuthName "localhost"
  AuthUserFile /usr/local/apache/.htpasswd
  Require valid-user
</Directory>

{{< /highlight >}}

这里 Apache 的认证是基于加密文件的认证，有点局限性。比如用户密码是存储在数据库中，这样的校验有点局限，当然可以通过扩展 C Apache 模块来解决（应该有类似的模块）。

本文尝试用 Nginx 来扩展一下 HTTP 401 认证，验证部分是由 lua-nginx-module 代码完成。

![](/images/nginx-lua-401-auth.png)

nginx.conf 配置片段：

{{< highlight bash >}}

location /ubuntu {
    access_by_lua '
        -- 用户和密码正确 access 则通过，并转向 proxy 部分。
        if ngx.var.remote_user == "smallfish" and ngx.var.remote_passwd == "12" then
            return
        end
        -- 返回 HTTP 401 认证输入框
        ngx.header.www_authenticate = [[Basic realm="Restricted"]]
        ngx.exit(401)
    ';
    proxy_pass http://127.0.0.1:8080/;
}

{{< /highlight >}}

这里的校验部分只是简单的字符串匹配，当然可以扩展成数据库方式（此示例不完整）：[http://forum.nginx.org/read.php?2,220692,220740](http://forum.nginx.org/read.php?2,220692,220740)

测试过程中刚好发现 [ngx_coolkit](https://github.com/FRiCKLE/ngx_coolkit) 模块在处理 $remote_passwd 一个 bug，此变量在 lua 中不能直接用 ngx.var.remote_passwd 读取，原因是 remote_passwd 变量在模块设置的标志位 NGX_HTTP_VAR_NOHASH，[@agentzh](http://weibo.com/agentzh) 已经在最新的 ngx_coolkit 中修复。

NGX_HTTP_VAR_NOHASH 完整说明参考微博：[http://weibo.com/1834459124/y4xzWzV2e](http://weibo.com/1834459124/y4xzWzV2e)

老版本的 ngx_coolkit 可以通过这样来绕过：

{{< highlight bash >}}

location / {
    set $passwd $remote_passwd;
    access_by_lua '
        if ngx.var.remote_user == "smallfish" and ngx.var.passwd == "12" then
            return
        end
        ...
}

{{< /highlight >}}

建议更新到最新版，或者直接安装 [ngx-openresty](http://openresty.org/) 包。


