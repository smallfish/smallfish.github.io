---
date: 2009-12-10
title: emacs windows配置笔记
---

俺最新.emcas配置地址是：[http://github.com/smallfish/editor/blob/master/emacs/.emacs](http://github.com/smallfish/editor/blob/master/emacs/.emacs)

最近手痒，看到不少牛x人物都是用emacs，遂在windows下装一个玩玩。

下载地址：[http://ftp.gnu.org/pub/gnu/emacs/windows/emacs-23.1-bin-i386.zip](http://ftp.gnu.org/pub/gnu/emacs/windows/emacs-23.1-bin-i386.zip)

解压到：D:\emacs-23.1

可以看到bin、etc、lisp等目录。主要运行都在bin目录。

runemacs.exe   这个就是运行文件拉，可以发送到桌面快捷方式。

可以运行一下addpm.exe这个，其作用就是把emacs加入到开始程序菜单里。

试着运行一下runemacs.exe，可以发现默认光标的位置，那是一个入门教程喔，还是中文的耶~

建议都看下这个入门的教程，常用的快捷键都有说明。

接下来加点常用的功能把，比如显示行号，goto line的功能。(俺也只配置了这个两项)

配置文件主要是_emacs或者.emacs，win下建议_emacs，点号开头的文件需要到cmd下才行。

_emacs文件放到哪儿呢？俺是直接修改了win注册表选项。

选项是：HKEY_CURRENT_USER\Software\GNU\Emacs，注意GNU\Emacs是需要新建的。接下来在Emacs里新建一个HOME项，值是你的emacs路径，比如我的：D:\emacs-23.1\bin。

然后需要做的就是把_emacs文件在这个bin目录下。

经过几经周折，显示行号和goto功能的配置如下：

{{< highlight python >}}
(global-linum-mode 1)
(global-set-key [(meta g)] 'goto-line)
{{< /highlight >}}

第一行是显示行号

第二行是设置meta+g转到goto功能，meta在windows下可以用alt来操作。

其他功能以后在后续补上，快捷键挺多，用了一会手指发酸。C-x 数字(1 2 3)挺好，可以开多窗口浏览了。

