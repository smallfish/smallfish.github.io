---
date: 2014-12-26
title: Docker Volume 属主设置
---

最近在测试 Volume 挂载时候有点问题，描述如下：

1. Docker 启动时候，设置挂载目录用户为 foo
2. 宿主机源目录属主为 smallfish

尝试了几种办法，比如：

1. container 启动后执行 `chown`
2. 构建镜像时候加入 `RUN chown foo /data`，这里不仅仅是这种尝试

不管是在 container 或者宿主机里进行设置，都会发现要么里面属主成数字或者宿主机的属主成数字。

原因无非是两个系统里用户 `uid/gid` 不一样，也可以很猥琐的在 container 里面 `adduser` 指定 `--uid` 参数。

百无聊赖的放狗继续搜啊搜，竟然搜到一篇：[Understanding Volumes in Docker](http://container-solutions.com/2014/12/understanding-volumes-docker/)

尝试了下，竟然可以：

{{< highlight bash >}}
RUN useradd foo
RUN mkdir /data && touch /data/a.txt
RUN chown foo:foo /data
VOLUME /data
{{< /highlight >}}

上面要注意 `VOLUME` 一定要放在最后，然后要先建立目录，再写个文件，文件随意。

重新构建之后，进入交互式：

{{< highlight bash >}}
# docker run -t -i -v /home/smallfish/xxx:/data testubuntu /bin/bash
root@527f8eceacca:/# ls -al /data
drwxr-xr-x  1 foo  staff  136 Dec 26 06:17 .
drwxr-xr-x 51 root root  4096 Dec 26 06:17 ..
drwxr-xr-x  1 foo  staff  170 Jul  4 15:57 xxx
{{< /highlight >}}

很惊喜的发现竟然属主变成 foo 了，我了个呵呵。。。

官方文档：[https://docs.docker.com/userguide/dockervolumes/](https://docs.docker.com/userguide/dockervolumes/)，其实也有提到一些描述，还是贴下之前参考那篇博客的原文：

    Docker is clever enough to copy any files that exist in the image under the volume mount into the volume and set the ownership correctly. This won’t happen if you specify a host directory for the volume (so that host files aren’t accidentally overwritten).



