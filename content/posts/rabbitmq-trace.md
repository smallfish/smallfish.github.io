---
date: 2012-12-19
title: RabbitMQ trace 日志调试
---

RabbitMQ 默认日志里只有类似客户端“accpet/close”等信息，对于有异常或者跟踪消息内部结构就比较麻烦了。

翻阅官方教程意外发现了 rabbitmq_tracing 插件和 [firehose](http://www.rabbitmq.com/firehose.html)。

注意：打开 trace 会影响消息写入功能，适当打开后请关闭。

自己顺手写了一个封装脚本，参考：[https://github.com/smallfish/rabbitmq-trace](https://github.com/smallfish/rabbitmq-trace)

安装上面的插件并开启 trace_on 之后，会发现多了两个 exchange：amq.rabbitmq.trace 和 amq.rabbitmq.log，类型均为：topic。

懂了吧，只要订阅这两个主题，就能收到：客户端连接、消息发收等具体信息了。

下面开始测试吧，先安装插件，并打开 trace_on：

{{< highlight bash >}}

$ sudo rabbitmq-plugins enable rabbitmq_tracing
$ （略去重启RabbitMQ命令）
$ sudo rabbitmqctl trace_on
Starting tracing for vhost "/" ...
...done.

{{< /highlight >}}

测试代码采用 Python 的 pika 库，片段如下：

{{< highlight bash >}}

import pika

def _on_message(ch, method, properties, body):
    ret = {}
    ret['routing_key'] = method.routing_key
    ret['headers'] = properties.headers
    ret['body'] = body
    print ret

conn = pika.BlockingConnection(pika.ConnectionParameters())
chan.queue_declare(exclusive=False, auto_delete=True) # 临时队列
queue = ret.method.queue
chan.queue_bind(exchange='amq.rabbitmq.trace', queue=queue, routing_key='#')
chan.queue_bind(exchange='amq.rabbitmq.log', queue=queue, routing_key='#')
chan.basic_consume(_on_message, queue=queue, no_ack=True)
chan.start_consuming()

{{< /highlight >}}

运行结果大致如下：

{{< highlight bash >}}

{'body': 'accepting AMQP connection <0.28967.1> (127.0.0.1:52930 -> 127.0.0.1:5672)\n', 
 'headers': None, 'routing_key': 'info'}
{'body': 'accepting AMQP connection <0.28967.1> (127.0.0.1:52930 -> 127.0.0.1:5672)\n', 
 'headers': {'node': 'rabbit@mac', 'exchange_name ': 'amq.rabbitmq.log', 'redelivered': 0, 
 'routing_keys': ['info'], 'properties': {'timestamp': 1355906883, 
 'content_type': 'text/plain'}}, 
 'routing_key': 'deliver.amq.gen-dJNnBwGUuigsfAzoUD9Zlw'}
{'body': '内容略去...', 'headers': {'node': 'rabbit@mac', 'exchange_name': 'amq.direct',
 'routing_keys': ['test2'], 'properties': {}}, 'routing_key': 'publish.amq.direct'}
{'body': 'closing AMQP connection <0.28967.1> (127.0.0.1:52930 -> 127.0.0.1:5672)\n', 
 'headers': None, 'routing_key': 'info'}
{'body': 'closing AMQP connection <0.28967.1> (127.0.0.1:52930 -> 127.0.0.1:5672)\n',
 'headers': {'node': 'rabbit@mac', 'exchange_name': 'amq.rabbitmq.log', 'redelivered': 0, 
 'routing_keys': ['info'], 'properties': {'timestamp': 1355906884, 
 'content_type': 'text/plain'}}, 
 'routing_key': 'deliver.amq.gen-dJNnBwGUuigsfAzoUD9Zlw'}

{{< /highlight >}}


