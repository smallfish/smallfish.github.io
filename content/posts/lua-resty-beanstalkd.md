---
date: 2012-11-25
title: lua-resty-beanstalkd 模块教程

tags:
- Lua
- Nginx
- Beanstalkd

---

本文涉及几个名词：

* ngx_lua [http://wiki.nginx.org/HttpLuaModule](http://wiki.nginx.org/HttpLuaModule)
    
     Embed the power of Lua into Nginx 摘自官方描述
         
* beanstalkd [http://kr.github.com/beanstalkd/](http://kr.github.com/beanstalkd/)

    一个轻量级的队列系统，实现了生产者消费者模型。协议类似 Memcached，相当简单易用。

* lua-resty-beanstalkd [https://github.com/smallfish/lua-resty-beanstalkd](https://github.com/smallfish/lua-resty-beanstalkd)

    基于 ngx_lua cosocket 模块封装，目前已支持 beanstalkd 协议有：put/reserve/delete/use/watch，其他协议持续更新中。类似的模块有：lua-resty-redis/memcached 等。

下面是读写的示例：

1. 生产者（发布消息），主要那应用了两个协议：use/put，代码如下：

{{< highlight bash >}}

location /t {
    content_by_lua '
        local beanstalkd = require "resty.beanstalkd"
        
        local bean, err = beanstalkd:new()

        local ok, err = bean:connect("127.0.0.1", 11300)
        if not ok then
            ngx.say("1: failed to connect: ", err)
            return
        end

        local ok, err = bean:use("default")
        if not ok then
            ngx.say("2: failed to use tube: ", err)
            return
        end
   
        local id, err = bean:put("hello")
        if not id then
            ngx.say("3: failed to put: ", err)
            return
        end

        ngx.say("put: ", id)

        bean:close()
    ';
}

{{< /highlight >}}

2. 消费者（消费消息），主要用了三个协议：watch/reserve/delete，代码如下：

{{< highlight bash >}}

location /t {
    content_by_lua '
        local beanstalkd = require "resty.beanstalkd"

        local bean, err = beanstalkd:new()

        local ok, err = bean:connect("127.0.0.1", 11300)
        if not ok then
            ngx.say("1: failed to connect: ", err)
            return
        end

        local ok, err = bean:watch("default")
        if not ok then
            ngx.say("2: failed to watch: ", err)
            return
        end

        local id, data = bean:reserve()
        if not id then
            ngx.say("3: failed to reserve: ", id)
            return
        else
            ngx.say("1: reserve: ", id)
            local ok, err = bean:delete(id)
            if not ok then
                ngx.say("4: failed to delete: ", id)
                return
            end
            ngx.say("2: delete: ", id)
        end

        bean:close()
    ';
}

{{< /highlight >}}

为了提升性能，可以调用 set_keepalive（提升50%以上性能），需要注意：

1. 代码中不要调用 xx:close 主动关闭连接。
2. 在对象使用后调用 xx:set_keepalive，而不是初始化 new/connect 时候调用此方法。



