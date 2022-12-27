---
date: 2011-09-15
title: auto-xhprof PHP自动性能测试工具
---

auto-xhprof 项目地址： [https://github.com/smallfish/auto-xhprof](https://github.com/smallfish/auto-xhprof)

最近需要对一些PHP应用和底层函数进行一些排错和性能方面的分析，不由得想起xhprof这个小强的利器。

当然也可以按照官方的示例来监控应用或者页面的数据，使用起来还是有点不爽。比如想监控所有访问超过2秒的页面性能情况，或者自动打开/关闭分析，请求url、响应时间等。

随手基于xhprof写了一个扩展小工具。

主要思路如下：

{{< highlight bash >}}
通过修改php.ini中的auto_prepend_file可以预加载auto-xhprof.php，自动打开xhprof功能。

;php.ini
auto_prepend_file = '/path/prepend.php'

; <?php
;    include_once '/path/auto-xhprof.php';
; ?>
{{< /highlight >}}

主要功能点有如下：

* 参数配置是否自动开启xhprof。
* 参数配置超时的阀值，比如2秒。
* 保存分析后的数据到MySQL中，供集中统一分析。
* 数据内容含：请求的URL、相应时间。
* 支持gearman异步保存数据。
* 提供封装的xhprof_start/xhprof_end对部分程序进行手动分析。
* 自动记录错误信息

图一（列表显示）：
![](/images/auto-xhprof-1.png)

图二（部分函数显示）：
![](/images/auto-xhprof-2.png)

源码文件说明：

{{< highlight bash >}}
auto-xhprof.php         全局加载文件。
auto-xhprof-config.php  全局配置文件，设置MySQL数据库和参数等。
gearman-worker.php      gearman后台处理worker进程。
web/                    web显示目录，xhprof列表页面和原xhprof展示部分
xhprof_lib/             xhprof库文件。
{{< /highlight >}}

--------- 扩展安装步骤 ---------

* 编译xhprof.so扩展
{{< highlight bash >}}
% cd <xhprof_source_directory>/extension/
% phpize
% ./configure --with-php-config=<path to php-config>
% make
% make install
{{< /highlight >}}

* 修改php.ini中extension，以支持xhprof扩展
{{< highlight bash >}}
[xhprof]
extension=xhprof.so
xhprof.output_dir=<directory> ; 需要有写入权限，可以写/tmp，或者不设置
{{< /highlight >}}

* 从github上下载auto-xhprof，并配置。
{{< highlight bash >}}
1. 非web目录拷贝到系统php include目录下，比如：/usr/include。
2. web目录拷贝到可访问的应用目录下。比如：/var/www/app1/web，前端访问为：http://app1/web/
3. 修改auto-xhprof-config.php中相应的数据库配置，超时阀值等参数。
{{< /highlight >}}

到这里，基本安装和配置结束了，访问web目录可以看到xhprof列表信息。

另外，如果支持gearman扩展的话，需要在php.ini中启动该扩展，并在auto-xhprof-config.php中配置：

{{< highlight bash >}}
define('__XHPROF_GERAMAN_SERVERS', '127.0.0.1:4730;127.0.0.1:4730'); // gearman 服务器定义

% gearmand -vvv -q libdrizzle --libdrizzle-host=127.0.0.1\
   --libdrizzle-user=root --libdrizzle-password=123456 --libdrizzle-db=gearman\
      --libdrizzle-table=queue --libdrizzle-mysql

% php gearman-worker.php
{{< /highlight >}}
