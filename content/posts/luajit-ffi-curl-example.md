---
date: 2012-10-11
title: LuaJIT FFI 调用 Curl 示例
---
[LuaJIT](http://luajit.org) 是一个好东西，比官方 [Lua](http://lua.org) 解释器性能上提升很多。ngx_lua/[ngx_openresty](http://openresty.org) 都推荐用 LuaJIT 来加速 Lua 代码。

除去性能和速度上的优势，LuaJIT 还提供了 C Binding 模块：FFI，可以理解为类似 Python 中的 ctypes 模块，不过更加小巧和直观。意味着可以很方便的调用动态库啦。

下面示例如何在 FFI 调用 Curl 中的相关函数，环境如下：

{{< highlight bash >}}

操作系统：   Mac OS X 10.8.2
LuaJIT版本：LuaJIT-2.0.0-beta10
Curl版本：  7.24.0

{{< /highlight >}}

测试代码如下：

{{< highlight bash >}}

local ffi = require 'ffi'

ffi.cdef[[
    void *curl_easy_init();
    int curl_easy_setopt(void *curl, int option, ...);
    int curl_easy_perform(void *curl);
    void curl_easy_cleanup(void *curl);
    char *curl_easy_strerror(int code);
]]

local libcurl = ffi.load('libcurl')

local curl = libcurl.curl_easy_init()
local CURLOPT_URL = 10002 -- 参考 curl/curl.h 中定义

if curl then
    libcurl.curl_easy_setopt(curl, CURLOPT_URL, 'http://example.com')
    res = libcurl.curl_easy_perform(curl)
    if res ~= 0 then
        print(ffi.string(libcurl.curl_easy_strerror(res)))
    end
    libcurl.curl_easy_cleanup(curl)
end

{{< /highlight >}}


参考：[http://develcuy.com/en/playing-luajit-ffi](http://develcuy.com/en/playing-luajit-ffi)（被此文坑了一下，示例中 curl_easy_setopt 函数 option 类型为：char，正确的应该是：int）

报错信息如下（可以辅助设置 VERBOSE（值为41） 选项调试）：

{{< highlight bash >}}

URL using bad/illegal format or missing URL

{{< /highlight >}}


