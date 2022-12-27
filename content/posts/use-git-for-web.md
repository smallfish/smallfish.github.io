---
date: 2009-12-08
title: 使用Git维护你的网站
---

<img class="alignnone size-full wp-image-92" title="git-tree" src="http://chenxiaoyu.org/wp-content/uploads/2009/12/git-tree.gif" alt="git-tree" width="263" height="316" />

简介

git是由[http://en.wikipedia.org/wiki/Linus_Torvalds](http://en.wikipedia.org/wiki/Linus_Torvalds)编写的一个开放源码的版本控制系统. 它的主要目标是高度分散, 效率超过其他竞争对手.

我就是使用git维护本网站. 我知道git不应该这样的粗重任务的使用, 即每一个开发者维护一份代码拷贝, 但是它工作的很好, 所有我使用它.

本文的目的就是说明如何在家里或者笔记本里维护一个本地拷贝, 然后让这些修改提交到互联网主机上. 下面就是介绍如何设置.

安装

{{< highlight python >}}
# Gentoo
emerge git
{{< /highlight >}}


{{< highlight python >}}
# Debian/Ubuntu
apt-get install git-core
{{< /highlight >}}


{{< highlight python >}}
# RedHat/Fedora
yum install git
{{< /highlight >}}

初始化

你会进入你的服务器的目录和初始化git仓库.

{{< highlight python >}}
# 进入你的web目录
cd /$wherever/html/
{{< /highlight >}}


{{< highlight python >}}
# 初始化仓库
git init
{{< /highlight >}}


{{< highlight python >}}
# 添加所有内容
git add .
{{< /highlight >}}


{{< highlight python >}}
# 提交 -m备注
git commit -a -m "The Initial Import."
{{< /highlight >}}

然后返回你的html父目录, 克隆新的git-ized web目录.

{{< highlight python >}}
# 返回你的html目录
cd ..
{{< /highlight >}}


{{< highlight python >}}
# 克隆你的web目录到 html.git
git clone --bare html html.git
{{< /highlight >}}

现在你已经初始化好了仓库, 并将整个目录(递归)到该库中, 并进行了初次提交, 为web目录建立了一个git克隆. 这个git目录(html.git)是整个过程的关键.

获取一份开发环境的拷贝

1. 转到你的开发系统
2. 安装git
3. 从你的开发目录运行下面命令

{{< highlight python >}}
git clone ssh://yoursite.com/path/to/html.git
{{< /highlight >}}

现在已经获得一个完整的网站服务器版本的本地拷贝.

使你的web目录克隆Git目录

记住, 你的html.git是这里的关键, 而不是现有的html目录, 所以你要切换出来, 备份目录, 然后:

{{< highlight python >}}
# 备份html目录, 然后克隆html.git
mv html html.backup; git clone html.git
{{< /highlight >}}

这在当前目录获取一份html.git的拷贝, 当然名字还是html. 这就是为什么备份旧的html目录.

自动推送修改

把你的html.git目录添加到post-update钩子中

{{< highlight python >}}
cd ../htdocs
env -i git pull
{{< /highlight >}}

修改钩子程序为可执行

{{< highlight python >}}
chmod +x post-update
{{< /highlight >}}

在你的开发环境的变动 

现在编辑的网站, 打开一个新的TextMate项目(你使用TextMate对吗?), 并拖动到html克隆目录. 整个结构都准备好了.

1. 通常的变动
2. 保存更改
3. 运行下面的程序, 例如(QuickSearchBox, TextMate等)

{{< highlight python >}}
# This is for an OS X box
cd /Users/daniel/Development/html/
git commit -a -m "Another update."
git push
{{< /highlight >}}

这基本上是更新到git仓库最重要的两个命令: commit(注意:你的标记(如果想回滚的话))和push(推送到服务器).

[根据你的操作系统和git安装, 你可能需要chmod +x 钩子程序, 然后再继续]

现在你只需要激活post-update 钩子程序, 将会自动的获取web目录.

如果你在服务器端的操作基本也是相同的git commit和git push, 然后你的开发环境下git pull同步备份就可以了. 当然你还可以使用脚本, 如果你需要的话.

原文参见: [http://danielmiessler.com/blog/using-git-to-maintain-your-website](http://danielmiessler.com/blog/using-git-to-maintain-your-website)

