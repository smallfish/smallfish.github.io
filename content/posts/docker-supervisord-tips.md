---
date: 2017-12-12
title: docker supervisord 管理多进程的一些建议

tags:
- Docker
- Supervisord

---


上一次提到 supervisord 还是 2011 年 <a href="/2011/05/31/python-supervisor/">python supervisor</a> 这里再补充一篇，基本上是这几年在 docker 里面的一些用法和建议。在容器里面是不太建议里面跑多个进程（服务）的玩法，但是有的时候实际业务在所难免


#### 配置使用绝对路径

启动参数 `-c` 指定配置 supervisord.conf 绝对路径。比如 Dockerfile 最后这么写：

```
ENTRYPOINT ["/usr/bin/supervisord", "-c", "/etc/supervisor/supervisord.conf"]
```

#### 多个启动项配置文件统一目录

如 `/etc/supervisor/conf.d/`，并且在 supervisord.conf include 这里目录

```
[include]
files = /etc/supervisor/conf.d/*.conf
```

#### 每个启动项日志统一

统一放入目录，如：`/tmp/xxx_stdout.log`，并设置日志滚动大小和保留份数

#### supervisord 配置

- 需要配置 `nodaemon=true`，默认输出扔给 docker
- supervisorctl 使用 `http server` 方式管理，这里涉及 docker overlayfs unix sockets 一个问题，无法管理启动项，不知道现在是否解决

```
[inet_http_server]
port=127.0.0.1:9001
username=user
password=123

[supervisorctl]
serverurl=http://127.0.0.1:9001
username=user
password=123
```

缺点就是多监听本地一个端口，并注意不能和应用端口冲突

#### 一次性程序

比如要执行初始化，可以：

```
[program:xxx]
command=sh /xxx.sh
startretries=0
autorestart=false
```

#### 传递环境变量

针对非 root 用户启动，并且需要读取容器启动时候环境变量（env）传递，新写 ENTRYPOINT 脚本中获取环境变量，写入文件，最后再运行 supervisord

```
$ cat Dockerfile
# ...
ENTRYPOINT ["/entrypoint.sh"]

$ cat /entrypoint.sh

# ...
# 获取环境变量，写入配置，或者拼接完整 command 
# ...

/usr/bin/supervisord -c xx.conf
```

#### 重启或者重加载配置

在容器内 `kill -HUP 1` 即可

