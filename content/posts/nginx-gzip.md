---
date: 2012-03-08
title: Nginx GZip 压缩

tags:
- Nginx

---

Nginx GZip 模块文档详见：[http://wiki.nginx.org/HttpGzipModule](http://wiki.nginx.org/HttpGzipModule)

常用配置片段如下：

{{< highlight html >}}

gzip             on;
gzip_comp_level  2;                        # 压缩比例，比例越大，压缩时间越长。默认是1
gzip_types       text/css text/javascript; # 哪些文件可以被压缩
gzip_disable     "MSIE [1-6]\.";           # 无视IE6这个笨蛋~

{{< /highlight >}}

其中 gzip_types 选项默认只压缩 text/html，源码见：

{{< highlight c >}}
src/http/modules/ngx_http_gzip_filter_module.c 行152：
    &ngx_http_html_default_types[0]

src/http/ngx_http.c 行77：
    ngx_str_t  ngx_http_html_default_types[] = {
        ngx_string("text/html"),
{{< /highlight >}}

常用的静态 type 有，看自己需要压缩的情况而定：

{{< highlight bash >}}

text/html
text/plain
text/css
application/x-javascript
text/javascript
application/xml

{{< /highlight >}}

OK，到这里基本服务端已经配置完毕，Nginx 只需要 reload 一下即可。

下面来测试一下，用 curl 来如何测试服务端已经开启 gzip（测试条件是默认gzip_types，即只压缩 text.html ，其他 type 未压缩）：

{{< highlight bash >}}

查看是否开启gzip，需要客户端加入："Accept-Encoding: gzip, deflate" 头信息。

$ curl -I -H "Accept-Encoding: gzip, deflate" "http://localhost/tag.php"

HTTP/1.1 200 OK
Server: nginx
Date: Thu, 08 Mar 2012 07:23:46 GMT
Content-Type: text/html
Connection: close
Content-Encoding: gzip

$ curl -I -H "Accept-Encoding: gzip, deflate" "http://localhost/style.css"

HTTP/1.1 200 OK
Server: nginx
Date: Thu, 08 Mar 2012 07:23:54 GMT
Content-Type: text/css
Connection: close
Last-Modified: Tue, 27 Dec 2011 10:00:51 GMT
ETag: "BC612352322D435769C4BDC03DDB2572"
Content-Length: 22834
{{< /highlight >}}

可以看出来了把。第二个示例没有被压缩。


