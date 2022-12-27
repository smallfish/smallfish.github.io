---
date: 2009-12-15
title: Memcached Java/Python Client API 共享

tags:
- Memcached

---

用Python写了一个计划任务，定时更新Memcached中一个key值，写的很happy，几分钟搞定。

然后在Java Servlet测试，代码写的也很happy，编译 - 刷新，一气呵成。

然后发现值一直是null，再tail日志看看，异常不断：

{{< highlight python >}}
com.danga.MemCached.MemCachedClient Mon Jul 20 09:37:04 CST 2009 - ++++ exception thrown while trying to get object from cache for key: test_num
 com.danga.MemCached.MemCachedClient Mon Jul 20 09:37:04 CST 2009 - 3
 com.danga.MemCached.NestedIOException: 3
 at com.danga.MemCached.MemCachedClient.get(MemCachedClient.java:1408)
 at com.danga.MemCached.MemCachedClient.get(MemCachedClient.java:1270)
{{< /highlight >}}

晕倒，记得以前为了让两个语言实现API读写共享，手动去修改了两个的API包，实现了中文互读写。难不成今儿个还要手动去搞一把？

然后手动试了下：

{{< highlight python >}}

shell> telnet xxxxxx 11211
get test_num
VALUE test_num 4 2
23
{{< /highlight >}}

经查证VALUE协议返回的是 key flags len \r\n value 这样的格式，大悟：原来flags不一样啊，Java里面对int型赋值以后flags是0，而Python里则不一样，两者序列化的东西不同啊。懒得去 折腾两者序列化有啥不同。来点直接的把。

然后打开Python Memcached API，大概578行_val_to_store_info方法里，可以看到flags部分，是根据变量类型进行定义的，isinstance(val, str) 如果是str则pass。

到这里就简单了，直接在py代码里：mc.set('test_num', str(num))

Java读取OK。

