---
date: 2011-02-19
title: 开源网页截屏工具 CutyCapt Linux 下安装和使用
---

目的是想在服务器端生成某个网页的缩略图。Google了好久，发现一个好开源东西：CutyCapt。

系统：CentOS 5.5
官网：[http://cutycapt.sourceforge.net/](http://cutycapt.sourceforge.net/)
依赖：QT [http://www.qtsoftware.com/](http://www.qtsoftware.com/)

首次是在Debian下测试的，很顺利。换了CentOS不是太爽。具体安装和使用步骤如下：

1. 下载RPM包

{{< highlight python >}}
(64位)
wget http://dl.atrpms.net/el5-x86_64/atrpms/testing/qt44-4.4.3-10_4.el5.x86_64.rpm
wget http://dl.atrpms.net/el5-x86_64/atrpms/testing/qt44-x11-4.4.3-10_4.el5.x86_64.rpm
wget http://dl.atrpms.net/el5-x86_64/atrpms/testing/qt44-devel-4.4.3-10_4.el5.x86_64.rpm

(32位)
wget http://ftp.riken.go.jp/Linux/atrpms/el5-i386/atrpms/testing/qt44-4.4.3-10_4.el5.i386.rpm
wget http://ftp.riken.go.jp/Linux/atrpms/el5-i386/atrpms/testing/qt44-x11-4.4.3-10_4.el5.i386.rpm
wget http://ftp.riken.go.jp/Linux/atrpms/el5-i386/atrpms/testing/qt44-devel-4.4.3-10_4.el5.i386.rpm
{{< /highlight >}}

2. 安装 qt-devel 依赖包

{{< highlight python >}}
yum install libXi-devel
yum install libXinerama-devel
{{< /highlight >}}

3. 安装 qt 相关

{{< highlight python >}}
rpm -ivh qt44-4.4.3-10*
rpm -ivh qt44-x11-4.4.3-10*
# rpm -e qt-devel --nodeps --allmatches
rpm -ivh qt44-devel-4.4.3-10*
{{< /highlight >}}

4. 修改 /etc/profile，最后并：source /etc/profile

{{< highlight python >}}
export QTDIR=/usr/lib64/qt44
export QTLIB=/usr/lib64/qt44/lib
export QTINC=/usr/lib64/qt44/include
export LD_LIBRARY_PATH=$QTDIR/lib:$LD_LIBRARY_PATH
export PATH=$QTDIR/bin:$PATH
{{< /highlight >}}

5. 安装 cutycapt

{{< highlight python >}}
svn co https://cutycapt.svn.sourceforge.net/svnroot/cutycapt
mv cutycapt/CutyCapt /usr/local/CutyCapt
cd /usr/local/CutyCapt
qmake
make
{{< /highlight >}}

6. 安装模拟 x-server 服务端

{{< highlight python >}}
wget http://www.flexthinker.com/wp-content/uploads/2009/11/xvfb-run.sh.txt
mv ./xvfb-run.sh.txt /usr/local/CutyCapt/xvfb-run.sh
chmod u+x /usr/local/CutyCapt/xvfb-run.sh
{{< /highlight >}}

7. 开始欢快的截图吧

{{< highlight python >}}
/usr/local/CutyCapt/xvfb-run.sh --server-args="-screen 0, 1024x768x24" /usr/local/CutyCapt/CutyCapt --url=http://www.163.com --out=163.jpg
{{< /highlight >}}

8. 如果看不到汉字或乱码，需要安装chinese字体

{{< highlight python >}}
yum install fonts-chinese
{{< /highlight >}}

9. 由于截屏的是整个网站的页面，只需要第一屏幕

{{< highlight python >}}
convert -crop 1024x768+0+0 163.jpg 1632.jpg
{{< /highlight >}}

10. 缩小图片

{{< highlight python >}}
convert -resize 40%x40% 1632.jpg 1632.jpg
{{< /highlight >}}


