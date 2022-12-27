---
date: 2009-12-16
title: C Apache Module 开发入门

tags:
- C
- Apache

---

前言：

扩展Apache模块开发网上大部分教程都是围绕Perl语言，老外的《Writing Apache Modules with Perl and C》可以算是经典之作了，可惜一直都是针对老版本开发，而且主力语言是Perl，C语言部分只是略有介绍。不过相比较而言用Perl来扩展模块功能确实比 C语言来的快速以及便捷多了，也简单容易。我自己也在工作里应用了一部分，主要是在防盗链上面写了两个简单都模块，可以参考我写的另外两篇文章：[http://chenxiaoyu.org/blog/archives/120](http://chenxiaoyu.org/blog/archives/120)。说了那么多题外话，回到正题，这里只是用C语言实现一个简单的hello模块，模块功能是查询MySQL自带mysql数据库里都user表。

系统环境：

ArchLinux Apache2.2 MySQL 5.0

具体开发步骤：

1. 利用Apache自带都apxs建立hello模块：

{{< highlight python >}}
[root#localhost] apxs -g -n hello
{{< /highlight >}}

这样就会在当前目录下新建一个hello模块的文件目录，可以看到里面有：Makefile mod_hello.c modules.mk这样的文件，具体apxs路径查询下本机apache/bin目录。

2. 预览下mod_hello.c，可以看到里面apxs自动帮你生成一堆代码了，我们需要的只是修改里面的代码部分，先简单都介绍下里面的函数说明。

include 部分就是引入了一些必要都头文件
hello_handler 这个就是hello模块都主体部分，所有的显示、处理请求什么的都在这里。
hello_register_hooks hello_module 这俩个是需要导出的函数所必须的，先可以不管他们，按照生成的不动即可。

3. 修改hello_handler函数，里面可以看到request_rec *r，r有很多函数和变量，具体要参见文档了。里面的ap_rputs是输出，可以简单的理解为把字符串输出到r。

{{< highlight python >}}
static int hello_handler(request_rec *r)
{
if (strcmp(r->handler, "hello")) { // 判断apache配置文件里handler是否等于hello，不是就跳过
          return DECLINED;
     }
     r->content_type = "text/html"; // 设置content-type
     if (!r->header_only)
          ap_rputs("The sample page from mod_hello.c\n", r); // 输出一段文字
     return OK;// 返回 200 OK状态
}
{{< /highlight >}}

增加#include "mysq.h"，查询需要用到这个头文件。
具体代码参见本文结尾部分。

4. 编译模块

{{< highlight python >}}
[root#localhost] apxs -c -a -i -I/usr/include/mysql/ -lmysqlclient mod_hello.c
{{< /highlight >}}

可以看到一堆编译指令，加上-I和-l是编译mysql必须的，编译完会自动在httpd.conf加上 LoadModule hello_module modules/mod_hello.so

5. 修改httpd.conf
<Location /hello>
SetHandler hello
</Location
6. 重启apache，访问http://localhost/hello，看是否成功。

=====================

完整代码：

{{< highlight python >}}
#include "httpd.h"
#include "http_config.h"
#include "http_protocol.h"
#include "ap_config.h"
/* 头文件，本文用到了ap_rprintf函数 */
#include "apr.h"
#include "apr_lib.h"
#include "apr_strings.h"
#include "apr_want.h"
#include "mysql.h"

/* 定义mysql数据变量 */
const char *host = "localhost";
const char *user = "root";
const char *pass = "smallfish";
const char *db    = "mysql";

/* The sample content handler */
static int hello_handler(request_rec *r)
{
    if (strcmp(r->handler, "hello")) {
        return DECLINED;
    }
    r->content_type = "text/html";
    /* 定义mysql变量 */
    MYSQL mysql;
    MYSQL_RES *rs;
    MYSQL_ROW row;
    mysql_init(&amp;mysql); /* 初始化 */
    if (!mysql_real_connect(&amp;mysql, host, user, pass, db, 0, NULL, 0)) {/* 连接数据库 */
        ap_rprintf(r, "<li>Error:%d %s</li>\n", mysql_errno(&amp;mysql), mysql_error(&amp;mysql));
        return OK;
    }
    char *sql = "select host,user from user order by rand()";
    if (mysql_query(&amp;mysql, sql)!=0) { /* 查询 */
        ap_rprintf(r, "<li>Error : %d %s</li>\n", mysql_errno(&amp;mysql), mysql_error(&amp;mysql));
        return OK;
    }
    rs = mysql_store_result(&amp;mysql); /* 获取查询结果 */
    while ((row = mysql_fetch_row(rs))) { /* 获取每一行记录 */
        ap_rprintf(r, "<li>%s - %s</li>\n", row[0], row[1]);
    }
    mysql_free_result(rs); /* 释放结果集 */
    mysql_close(&amp;mysql); /* 关闭连接 */
    return OK;
}

static void hello_register_hooks(apr_pool_t *p)
{
    ap_hook_handler(hello_handler, NULL, NULL, APR_HOOK_MIDDLE);
}

/* Dispatch list for API hooks */
module AP_MODULE_DECLARE_DATA hello_module = {
    STANDARD20_MODULE_STUFF,
    NULL,                            /* create per-dir              config structures */
    NULL,                            /* merge  per-dir              config structures */
    NULL,                            /* create per-server config structures */
    NULL,                            /* merge  per-server config structures */
    NULL,                            /* table of config file commands                 */
    hello_register_hooks  /* register hooks                                */
};
{{< /highlight >}}


