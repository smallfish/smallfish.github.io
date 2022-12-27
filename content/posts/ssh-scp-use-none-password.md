---
date: 2009-12-16
title: SSH, SCP 不输入密码

tags:
- Linux
- SSH

---

经常在不同linux机器之间互相scp拷文件，每次总是要输入密码才可行。

通过ssh-keygen生成公钥，在两台机器之间互相建立信任通道即可。假设本地机器client，远程机器为server。

1. 生成rsa keygen

{{< highlight python >}}
[winter@client winter] $ ssh-keygen -b 1024 -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/winter/.ssh/id_rsa): <Enter>
Enter passphrase (empty for no passphrase): <Enter>
Enter same passphrase again: <Enter>
Your identification has been saved in /home/winter/.ssh/id_rsa.
Your public key has been saved in /home/winter/.ssh/id_rsa.pub.
The key fingerprint is:
33:d4:7b:9c:87:04:cf:14:40:22:6d:c7:15:78:97:6a winter@client
{{< /highlight >}}

直接上面公钥和私钥存放地址可以直接回车，私钥密码可以直接回车，也可以输入。

2. 查看.ssh目录下了多私钥和公钥文件

{{< highlight python >}}
[winter@client winter] $ ls .ssh/
id_rsa  id_rsa.pub  known_hosts
{{< /highlight >}}

3. 拷贝公钥到目标机器上，并改名成authorized_keys

首次scp命令时候还是会提示输入密码，还有是否继续链接的提示，以后

4. 测试ssh进入

{{< highlight python >}}
[winter@client winter] $ ssh 192.168.0.110
{{< /highlight >}}

5. ok，搞定！

{{< highlight python >}}
[winter@server winter] # it's ok!
{{< /highlight >}}


