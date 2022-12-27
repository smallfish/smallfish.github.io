---
date: 2011-11-09
title: Nginx Session 模块
---
上一篇[《Nginx第三方模块》](http://chenxiaoyu.org/2011/10/30/nginx-modules.html)涉及了数据库、Memcached以及Lua的扩展，但是相对于Web开发是不是还缺点什么呢？答案是回话（Session）模块。

这里还是需要感谢一下[@agentzh](http://weibo.com/agentzh)，已经封装好了encrypted-session模块。模块依赖ngx_devel_kit包。模块地址如下：

* [ngx_devel_kit](https://github.com/simpl/ngx_devel_kit)
* [encrypted-session-nginx-module](https://github.com/agentzh/encrypted-session-nginx-module)

编译很简单，类似如下：

{{< highlight bash >}}
./configure --prefix=/opt/nginx \
    --add-module=../ngx_devel_kit \
    --add-module=../encrypted-session-nginx-module
{{< /highlight >}}

重新编译Nginx二进制，Nginx需要quit再启动。而普通配置更新则reload即可：

{{< highlight bash >}}
1. kill -HUP `cat /path/nginx/logs/nginx.pid`
2. /path/nginx/sbin/nginx -s reload
{{< /highlight >}}

在测试之前，需要配置encrypted_session_key（长度32位）和encrypted_session_iv（长度16位）。

{{< highlight bash >}}
encrypted_session_key "abcdefghijklmnopqrstuvwxyz123456";
encrypted_session_iv "1234567812345678";
encrypted_session_expires 5; # 默认过期时间是1d（一天）
{{< /highlight >}}

话不到多说，直接来读写示例：

* 写入session，测试session名为name，值是smallfish。

{{< highlight bash >}}
location /session-write {
    set $name 'smallfish';
    set_encrypt_session $session_name $name;
    set_encode_base32 $session_name;
    add_header "Set-Cookie" "name=$session_name";
    echo "write name: $session_name";
}
{{< /highlight >}}

* 读取session，Nginx读取Cookie方式为：$cookie_xxx。xxx为cookie的名称。

{{< highlight bash >}}
location /session-read {
    set_decode_base32 $session_name $cookie_name;
    set_decrypt_session $name $session_name;
    echo "read name: $name";
}
{{< /highlight >}}



