---
date: 2011-11-24
title: JSON 美化输出
---

经常会碰到一些返回 JSON 格式的应用，默认都是一大坨字一起显示，完全是虐待自己的眼睛。

比如以下JSON：

{{< highlight bash >}}
mac:~ smallfish$ curl http://127.0.0.1:8108/postgres
[{"datname":"postgres","datdba":"10","encoding":6,"datcollate":"zh_CN.UTF-8","datctype":"zh_CN.UTF-8",
"datistemplate":false,"datallowconn":true,"datconnlimit":-1,"datlastsysoid":"12172","datfrozenxid":"985",
"dattablespace":"1663","datacl":null},{"datname":"smallfish","datdba":"16384","encoding":6,
"datcollate":"zh_CN.UTF-8","datctype":"zh_CN.UTF-8","datistemplate":false,"datallowconn":true,
"datconnlimit":-1,"datlastsysoid":"12172","datfrozenxid":"985","dattablespace":"1663","datacl":null}]
{{< /highlight >}}

顺手 Google 了下，发现了两种浏览方式：命令行查看和浏览器查看。

一、浏览器查看方式，需要安装 Chrome 插件：[JSONView](https://chrome.google.com/webstore/detail/chklaanhfefbnpoihckbnefhakgolnmc)。

![](/images/chrome-jsonview.png)

二、命令行查看，需要安装 Python。

{{< highlight bash >}}
mac:~ smallfish$ curl http://127.0.0.1:8108/postgres | python -mjson.tool 
{{< /highlight >}}

![](/images/python-json-format.png)

是不是美观很多了？


