---
date: 2011-10-30
title: Nginx 第三方模块试用记

tags:
- Nginx

---

最近试用了几个[@agentzh](http://weibo.com/agentzh)写的第三方[Nginx](http://nginx.net)模块，甚为愉悦，没想到在Nginx可以玩很多技巧和扩展，分享一下。

本文尝试的几个模块大概分为：

* echo
* memcached
* nginx
* lua

详细模块地址分别为：

* ngx_devel_kit <https://github.com/simpl/ngx_devel_kit>
* set-misc-nginx-module <https://github.com/agentzh/set-misc-nginx-module>
* memc-nginx-module <https://github.com/agentzh/memc-nginx-module>
* echo-nginx-module <https://github.com/agentzh/echo-nginx-module>
* lua-nginx-module <https://github.com/chaoslawful/lua-nginx-module>
* srcache-nginx-module <https://github.com/agentzh/srcache-nginx-module>
* drizzle-nginx-module <https://github.com/chaoslawful/drizzle-nginx-module>
* rds-json-nginx-module <https://github.com/agentzh/rds-json-nginx-module>

为了省事这里一股脑把上面的module全部下载好，一起编译。PS：如果更懒惰的可以尝试下[openresty](http://openresty.org)项目，它帮你打包好Nginx和一堆扩展模块，得感谢@agentzh。

这里编译和drizzle和lua模块，在编译Nginx之前需要设置一下这两个库的LIB和INCLUDE文件地址：

{{< highlight bash >}}
-- lua --
export LUA_LIB=/path/to/lua/lib
export LUA_INC=/path/to/lua/include

-- drizzle --
export LIBDRIZZLE_INC=/opt/drizzle/include/libdrizzle-1.0
export LIBDRIZZLE_LIB=/opt/drizzle/lib
{{< /highlight >}}

Nginx编译选项如下，请注意先后顺序：

{{< highlight bash >}}
./configure --prefix=/opt/nginx \
    --with-pcre=../pcre \
    --add-module=../ngx_devel_kit \
    --add-module=../set-misc-nginx-module \
    --add-module=../memc-nginx-module \
    --add-module=../echo-nginx-module \
    --add-module=../lua-nginx-module \
    --add-module=../srcache-nginx-module \
    --add-module=../drizzle-nginx-module \
    --add-module=../rds-json-nginx-module
{{< /highlight >}}

重新编译Nginx二进制，Nginx需要quit再启动。而普通配置更新则reload即可：

{{< highlight bash >}}
1. kill -HUP `cat /path/nginx/logs/nginx.pid`
2. /path/nginx/sbin/nginx -s reload
{{< /highlight >}}

PS：详细的模块编译可以参考各自模块的文档说明。

好吧，假设到这里已经安装和编译好Nginx以及一堆第三方扩展。下面开始尝试上面那些有趣的模块~

**场景1：Helloworld**

程序里helloworld很简单，nginx的hello如何实现呢？

{{< highlight bash >}}
1. 最简单的helloworld
location /hello1 {
    echo "hello 1111!";
}

2. 异步请求其他echo请求
location /hello2 {
    echo "hello 2222!";
    echo_location_async /hello1;
}

3. 输出GET请求参数，假设参数名是name，这里并对name参数进行解码
location /hello3 {
    set_unescape_uri $name $arg_name;
    set_if_empty $name "None";
    echo "hello, $name!";
}

是不是很帅气有趣。例2中的subrequest是完全non-blocking的。
echo模块还有timer扩展，页面输出加载时间也是小case了。
{{< /highlight >}}

**场景2：Memcached HTTP 接口**

想当年哥为了兼容不同语言的api，自己封装了memcached操作，真是挺苦逼的，当时还非常羡慕tokyotyrant的http接口来着，现在用memc模块效果一样，还有upstream和keepalive功能哦~

{{< highlight bash >}}
location /memcached {
    set $memc_cmd $arg_cmd;
    set $memc_key $arg_key;
    set $memc_value $arg_val;
    set $memc_exptime $arg_exptime;
    memc_pass 127.0.0.1:11211;
}

URL访问类似 http://host:port/memcached?cmd=aaa&key=bbb&val=ccc

还需要自己封装么？完全的基于url编程了，不再考虑各自语言的memcached api，cluster也简化了。
{{< /highlight >}}

**场景3：数据库查询**

这里用了libdrizzle模块，兼容MySQL、Drizzle、SQLite数据库。有时候为了一些查询或者接口，还需要用语言来封装一些数据库查询操作，现在直接通过drizzle模块可以很轻松的做到~~

{{< highlight bash >}}
# 添加MySQL配置
upstream mysql {
    drizzle_server 127.0.0.1:3306 dbname=test user=smallfish password=123 protocol=mysql;
}

# 通过url匹配出name，并编码防止注入，最后以json格式输出结果。
location ~ '^/mysql/(.*)' {
    set $name $1;
    set_quote_sql_str $quote_name $name;
    set $sql "SELECT * FROM users WHERE name=$quote_name";
    drizzle_query $sql;
    drizzle_pass mysql;
    rds_json on;
}

# 查看MySQL服务状态，这个很实用哦。
location /mysql-status {
    drizzle_status;
}
{{< /highlight >}}

**场景4：lua扩展**

内嵌lua语言，超级牛力哦~~~想干啥都可以了，而且还可以充分发挥lua coroutine的风骚，淘宝量子统计完全是lua扩展。

{{< highlight bash >}}
location /lua1 {
    default_type 'text/plain';
    content_by_lua 'ngx.say("hello, lua")';
}

# 请求另外的url
location /lua2 {
    content_by_lua '
        local res = ngx.location.capture("/hello1")
        ngx.say("data: " .. res.body)
     ';
}

Lua支持的选项很多，具体可参考官网WIKI文档。
{{< /highlight >}}

嗯，这里尝试玩了几个模块，详细的例子可以参考各自的模块文档，里面都有详细的说明和选项配置。

可以看出和传统编程有些微不同之处，完全是基于URL的HTTP接口，比传统的方式更为简单，更为清晰。还有non-blocking的实现，更见轻便。


