---
date: 2009-12-16
title: Java调用Linux SCP操作

tags:
- Linux

---

先来回顾下linux下scp命令的用法：

{{< highlight python >}}
[shell $] scp -r /本地目录或文件 user@192.168.0.110:/远程目录
{{< /highlight >}}

这条命令是把本地的目录或者文件拷贝到远程192.168.0.110一个目录下，如果是从远程拷到本地，则反一下ip和目录。-r则是递归目录。更多参见scp --help

最近在Java里调用scp，是通过一个JSP页面来触发。为了在调用系统命令时候不出现提示密码，两台机器配置好了信任关系，可以参考[http://chenxiaoyu.org/blog/archives/133](http://chenxiaoyu.org/blog/archives/133)，大致代码如下：

{{< highlight python >}}
Runtime.getRuntime().exec("scp /aa.txt root@192.168.0.110:/bb");
{{< /highlight >}}

try时候也没任何异常，但是文件没拷贝过去，最后根据Process的waitFor()获取命令返回值是1。

这下可以肯定的是调用系统命令失败，在System.out.println里打印出command，linux下运行是没错的。为何呢？

后来发现原来是用户权限的问题，默认apache运行用户是nobody，根本没权限调用scp命令，配置的信任关系也是本机的root用户。

那就重新加一个user把，adduser...到配置好信任关系，在scp -i 指定一个rsa文件，并把rsa文件复制到/tmp目录下，权限为0755，继续刷新，后台可以看到提示输入密码之类的output了。

貌似还比较棘手，最后还是搜了下，发现有关Java scp的库，[http://www.ganymed.ethz.ch/ssh2/](http://www.ganymed.ethz.ch/ssh2/)。貌似比较老，先来测试一下把。

{{< highlight python >}}
Connection conn = new Connection(“192.168.0.110”);
conn.connect();
boolean isAuthenticated = conn.authenticateWithPassword(“root”, "***********");
if (isAuthenticated == false)
    throw new IOException("Authentication failed.");
SCPClient client = new SCPClient(conn);
client.put("/aa.txt", "/bb");
conn.close();
{{< /highlight >}}

OK！发现竟然可以一次运行了。算了就不调用系统命令了，直接使用这个库把。

client.put方法第一个参数可以是个数组，即文件名的数组。暂时没找到整个目录的方法，就自己手动获取下目录文件列表把。

