---
date: 2010-05-10
title: "[图文解说] Virtual Box 通过 NAT(默认) 共享虚拟机中的服务"
---

Virtual Box 是个不错的虚拟机，小巧，功能也齐全。好像有点推销鸟。说正题，上次有个朋友就提到怎么能主机里访问虚拟机里的服务，昨晚实验了下，颇为顺利。记录下。这里利用的是默认的NAT上网，也就是共享主机上网，而不是设置独立的IP。

主机：Win XP
虚拟：Ubuntu 9.10

目的：Win里ssh进Ubuntu，能访问里面提供的服务。

主要是通过 VBoxManage setextradata 设置一些属性。

先上几个步骤图把。注意一下修改，先得关闭虚拟机，修改完事以后再启动。

1. 查看虚拟机中的名称：ubuntu9

2. 进入本机Vbox目录，运行VBoxManage，查看下。

 3. 添加三个项目

下面的pcnet是vbox里的网络设置，0是表示第一个网卡，后面一次类推。22是ssh端口，映射到主机的22端口。

{{< highlight python >}}
VBoxManage setextradata "ubuntu9"  "VBoxInternal/Devices/pcnet/0/LUN#0/Config/guestssh/Protocol" TCP
VBoxManage setextradata "ubuntu9"  "VBoxInternal/Devices/pcnet/0/LUN#0/Config/guestssh/GuestPort" 22
VBoxManage setextradata "ubuntu9"  "VBoxInternal/Devices/pcnet/0/LUN#0/Config/guestssh/HostPort" 22
{{< /highlight >}}

5. 启动虚拟机。

6. 设置putty登陆之。


到这里已经顺利ssh 到127.0.0.1，如果想访问虚拟机里的web服务器呢？同样很简单。

只要如下设置，web端口为81，跟上面也雷同：

{{< highlight python >}}
VBoxManage setextradata "ubuntu9" "VBoxInternal/Devices/pcnet/0/LUN#0/Config/web/Protocol" TCP
VBoxManage setextradata "ubuntu9" "VBoxInternal/Devices/pcnet/0/LUN#0/Config/web/GuestPort" 81
VBoxManage setextradata "ubuntu9" "VBoxInternal/Devices/pcnet/0/LUN#0/Config/web/HostPort" 81
{{< /highlight >}}

另外如果想清空上面设置的选项，只要不设置后面的值即可：

{{< highlight python >}}
VBoxManage setextradata "ubuntu9" "VBoxInternal/Devices/pcnet/0/LUN#0/Config/web/Protocol"
{{< /highlight >}}


