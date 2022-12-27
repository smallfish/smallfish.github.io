---
date: 2009-12-15
title: Pexpect通过SSH执行远程命令

tags:
- Linux
- Python

---

pexpect是python一个模块，可以通过：easy_install pexpect 来安装。

这里主要是用pexpect执行ssh，查看远程uptime和df -h看硬盘状况。

{{< highlight python >}}
#ssh_cmd.py
#coding:utf-8
import pexpect

def ssh_cmd(ip, user, passwd, cmd):
    ssh = pexpect.spawn('ssh %s@%s "%s"' % (user, ip, cmd))
    r = ''
    try:
        i = ssh.expect(['password: ', 'continue connecting (yes/no)?'])
        if i == 0 :
            ssh.sendline(passwd)
        elif i == 1:
            ssh.sendline('yes')
    except pexpect.EOF:
        ssh.close()
    else:
        r = ssh.read()
        ssh.expect(pexpect.EOF)
        ssh.close()
    return r

hosts = '''
192.168.0.12:smallfish:1234:df -h,uptime
192.168.0.13:smallfish:1234:df -h,uptime
'''

for host in hosts.split("\n"):
    if host:
        ip, user, passwd, cmds = host.split(":")
        for cmd in cmds.split(","):
            print "-- %s run:%s --" % (ip, cmd)
            print ssh_cmd(ip, user, passwd, cmd)
{{< /highlight >}}

hosts数组格式是：主机IP:用户名:密码:命令 (多个命令用逗号, 隔开)
可以看出打印出相应的结果了，可以拼成html发送mail看起来比较美观些咯！

