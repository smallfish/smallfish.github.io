---
date: 2018-01-18
title: wrk 使用

tags:
- Linux
- Wrk

---

wrk 使用小记，备忘用

### 简述

主要用于 HTTP 类接口压测，作为 ab（Apache Bench）替代品，支持脚本方式进行扩展

安装过程这里不再赘述，运行方式可以：

- 安装 wrk 命令行 https://github.com/wg/wrk
- docker 运行 `docker pull williamyeh/wrk`

### 简单使用

常用参数包括：

- -t，线程数，建议机器核数
- -d，测试时间，默认单位秒（s），支持 s/m/h 单位，比如 1m
- -c，打开连接数，需要大于 -t 参数数值
- -s，脚本（lua）文件地址
- --latency，打印延时统计信息


简单使用：

```bash
$ wrk -d10m -t24 -c100 --script=write.lua \
  --latency "http://127.0.0.1:8080/xx"
Running 10m test @ http://127.0.0.1:8080/xx
  24 threads and 100 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     9.02ms   57.73ms   1.04s    98.13%
    Req/Sec     1.81k   217.80     2.76k    89.74%
  Latency Distribution
     50%    1.93ms
     75%    2.78ms
     90%    4.33ms
     99%  245.27ms
  25472002 requests in 10.00m, 5.88GB read
Requests/sec:  42447.34
Transfer/sec:     10.04MB
```

类似如此的结果，需要关注的无非是：req/sec，latency 分布

或者：

```bash
$ docker run -it --rm --net host williamyeh/wrk \
  -d10m -t24 -c100 --latency "http://127.0.0.1:8080/xx"
...
```

### 扩展

除去基本使用，还可以使用脚本（lua）来实现一些定制的需求，能 hook 扩展点包括：

- setup(thread)，线程只执行一次
- init(args)，线程只执行一次
- delay()，每次请求调用，可以延迟多少 ms
- request()，每次请求调用
- response(status, headers, body)，每次请求调用
- done(summary, latency, requests)，整个过程结束一次

变量结构：

```lua
wrk = {
    scheme  = "http",
    host    = "localhost",
    port    = nil,
    method  = "GET",
    path    = "/",
    headers = {},
    body    = nil,
    thread  = <userdata>,
}
```

官方源码 https://github.com/wg/wrk/tree/master/scripts 这里有一些简单的例子可以参考

1.POST 请求

```lua
wrk.method = 'POST'
wrk.body = 'xxx'
```

2.随机请求

```lua
math.randomseed(os.time())

request = function()
    n = math.random(10, 20)
    return wrk.format('POST', '/xx', nil, 'xx-' .. n)
end

-- wrk.format(method, path, headers, body)
```

3.header 添加

```lua
wrk.headers['X-Ha'] = 'xxx'
```

