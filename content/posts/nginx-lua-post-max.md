---
date: 2012-01-04
title: Nginx-Lua过滤POST请求
---

注：此文章会持续更新

2012 来的几天关于Hash攻击的文章不断，基本语言级别的都收到影响。

看了下 PHP 相关 patch，基本就是对 POST 的 key 数量做一个限制，其他提供的 patch 差不多也是如此。

刚好可以尝试一下 [nginx-lua](https://github.com/chaoslawful/lua-nginx-module) 模块，这里简单贴一些代码，编译步骤就略去。

本文只是根据 POST 参数个数进行简单校验的测试。

这里大概有几个步骤：

加载 conf/post-limit.lua，文件内容在下一段

{{< highlight bash >}}

access_by_lua_file 'conf/post-limit.lua';

{{< /highlight >}}

conf/post-limit.lua 文件内容：

{{< highlight bash >}}

ngx.req.read_body()

local method   = ngx.var.request_method
local max_size = 2                               -- 参数最多个数，这里测试用，2个

if method == 'POST' then                         -- 只过滤 POST 请求
    local data = ngx.req.get_body_data()
    if data then
        local count = 0
        local i     = 0
        while true do
            if count > max_size then             -- 大于2次，重定向错误页面
                ngx.redirect('/post-max-count')
            end
            i = string.find(data, '&', i+1)      -- 查找 & 字符
            if i == nil then break end
            count = count + 1
        end
    else
        ngx.redirect('/post-error')
    end
end

{{< /highlight >}}

原先 ngx.req.get_post_args 函数来判断，发现也收到 hash 的影响，最后采取原始的循环字符串版本。

完整 nginx 配置片段：

{{< highlight bash >}}

location /test {
    access_by_lua_file 'conf/post-limit.lua';
    root html;
}

{{< /highlight >}}

reload nginx 即可。可以这样来访问测试：

{{< highlight bash >}}

$ curl --data "a=1&a=11&b=2" http://localhost/test/1.html
返回 405，可以正常 GET 页面内容。

$ curl --data "a=1&a=11&b=2&c=1" http://localhost/test/1.html
返回 302，重定向到 /post-max-error 错误。

{{< /highlight >}}

想起之前用 ModPerl 重写了 Apache 的 TransHandler/AccessHandler，跟这个比较类似，加一些过滤器。

感谢 [@agentzh](http://weibo.com/agentzh) 建议，需要注意一下，在 Nginx 配置中 client_max_body_size 和 client_body_buffer_size 需要设为相同大小。



