---
date: 2013-03-28
title: RabbitMQ REST API
---

最近造了一个轮子：[rabbitmq-http](https://github.com/smallfish/rabbitmq-http)，源于内部项目的一个翻版。基于 [Go](http://golang.org) 语言实现。

先说说为什么要写 HTTP API，在使用 RabbitMQ 过程中碰到了以下几个问题：

* 多语言，这样就要求每个语言都需要安装对应的 amqp 库。
* 版本，“历史遗留”原因，对应的 amqp 库比较古老，升级不易。
* 协议实现不全，这个也是跟版本有点关系。
* 切分和扩展，想自己逻辑里按照用户或 vhost 等来做 hash 切分有点麻烦。
* TCP，调试和跟踪不易啊。
* 功能复杂，对于使用者来说曲线较高，大都数情况只是生产者消费者模型。

其实也不算是问题，就是不爽的地方。也就是想找一个自由度略大的方案，然后使用起来也简单方便。

就整出了一个 HTTP 的方案，刚好最近手痒，想用 Go 语言写点东西，遂趁着周末和晚上整了一下。

仓库地址：[https://github.com/smallfish/rabbitmq-http](https://github.com/smallfish/rabbitmq-http)

如何安装依赖、编译和启动这些操作，这里就不再赘述，直接参考项目里的 [README.md](https://github.com/smallfish/rabbitmq-http/blob/master/README.md)

目前封装的接口大致分为一下几类：

* Exchange
    * 新建
    * 删除
* Queue
    * 新建
    * 删除
    * 绑定/取消绑定
    * 读取消息
* Message
    * 发布新消息

上面的新建、删除、发送或者读取的行为对应 HTTP 请求中的几种方法，比如：POST、DELETE、GET等。

整个 API 的返回值（HTTP状态码）有一下三类：

* 200，操作 OK，返回的 Body 体中有简单描述字符。
* 405，不支持的请求方式，比如接口尚未实现。
* 500，服务器端错误，Body 体中会有错误的描述。比如队列不存在、删除错误等。

好了，还是来一点实际的例子吧。完全是 curl 操作：

新建一个 Exchange：

{{< highlight bash >}}

$ curl -i -X POST http://127.0.0.1:8080/exchange -d \
'{"name": "e1", "type": "topic", "durable": true, "autodelete": false}'

{{< /highlight >}}
    
新建一个 Queue：

{{< highlight bash >}}

$ curl -i -X POST http://127.0.0.1:8080/queue -d \
'{"name": "q1"}'

{{< /highlight >}}

绑定 Queue 到 Exchange 上：

{{< highlight bash >}}

$ curl -i -X POST http://127.0.0.1:8080/queue/bind -d \
'{"queue": "q1", "exchange": "e1", "keys": ["aa", "bb", "cc"]}'

{{< /highlight >}}

发布消息到 Exchange，指定 Routing-key：

{{< highlight bash >}}

$ curl -i -X POST http://127.0.0.1:8080/publish -d \
'{"exchange": "e1", "key": "bb", "body": "hahaha"}'

{{< /highlight >}}

读取一下刚才发送的消息，即消费某个 Queue：

{{< highlight bash >}}

$ curl -i "http://127.0.0.1:8080/queue?name=q1"
        
PS：消费的接口输出为 Chunked 模式，可以用类似文件的行读取方式，接口是 HTTP 长连接。
同时这个接口也支持多个 Queue 一起消费，类似：“/queue?name=q1&name=q2” 即可。

{{< /highlight >}}


基本常用的接口已经暴露出来， 后续还会封装一些其他语言的 SDK，其实都是普通的 HTTP 的请求。

欢迎 fork，欢迎当小白鼠， ^_^


