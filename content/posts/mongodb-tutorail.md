---
date: 2010-04-27
title: 【译】MongoDB 入门教程
---

原文参见：[http://www.mongodb.org/display/DOCS/Tutorial](http://www.mongodb.org/display/DOCS/Tutorial)

启动数据库

下载 [http://www.mongodb.org/display/DOCS/Downloads](http://www.mongodb.org/display/DOCS/Downloads), 解压后并启动:

{{< highlight python >}}
$ bin/mongod
{{< /highlight >}}

MongoDB 默认存储数据目录为 /data/db/ (或者 c:\data\db), 当然你也可以修改成不同目录, 只需要指定 --dbpath 参数:

{{< highlight python >}}
$ bin/mongod --dbpath /path/to/my/data/dir
{{< /highlight >}}

获取数据库连接

现在我们就可以使用自带的shell工具来操作数据库了. (我们也可以使用各种编程语言的驱动来使用MongoDB, 自带的shell工具可以方便我们管理数据库)

启动 MongoDB JavaScript 工具:

{{< highlight python >}}
$ bin/mongo
{{< /highlight >}}

默认 shell 连接的是本机localhost 上面的 test库, 会看到:

{{< highlight python >}}
MongoDB shell version: 0.9.8
url: test
connecting to: test
type "help" for help
>
{{< /highlight >}}

"connecting to:" 这个会显示你正在使用的数据库的名称. 想换数据库的话可以:

{{< highlight python >}}
> use mydb
{{< /highlight >}}

可以输入 help 来查看所有的命令.

插入数据到集合

下面我们来建立一个test的集合并写入一些数据. 建立两个对象, j 和 t , 并保存到集合中去.
在例子里  '>'  来表示是 shell 输入提示符

{{< highlight python >}}
> j = { name : "mongo" };
{"name" : "mongo"}
> t = { x : 3 };
{ "x" : 3  }
> db.things.save(j);
> db.things.save(t);
> db.things.find();
{"name" : "mongo" , "_id" : ObjectId("497cf60751712cf7758fbdbb")}
{"x" : 3 , "_id" : ObjectId("497cf61651712cf7758fbdbc")}
>
{{< /highlight >}}

有几点需要注意下 :
<ul>
	<li>不需要预先建立一个集合. 在第一次插入数据时候会自动建立.</li>
	<li>在例子其实可以存储任何结构的数据, 当然在实际应用我们存储的还是相同元素的集合. 这个特性其实可以在应用里很灵活, 你不需要类似 alter table 来修改你的数据结构</li>
	<li>每次插入数据时候对象都会有一个ID, 名字叫 _id.</li>
</ul>
当你运行不同的例子, 你的对象ID值都是不同的.

下面再加点数据:

{{< highlight python >}}
> for( var i = 1; i < 10; i++ ) db.things.save( { x:4, j:i } ); > db.things.find();
{"name" : "mongo" , "_id" : ObjectId("497cf60751712cf7758fbdbb")}
{"x" : 3 , "_id" : ObjectId("497cf61651712cf7758fbdbc")}
{"x" : 4 , "j" : 1 , "_id" : ObjectId("497cf87151712cf7758fbdbd")}
{"x" : 4 , "j" : 2 , "_id" : ObjectId("497cf87151712cf7758fbdbe")}
{"x" : 4 , "j" : 3 , "_id" : ObjectId("497cf87151712cf7758fbdbf")}
{"x" : 4 , "j" : 4 , "_id" : ObjectId("497cf87151712cf7758fbdc0")}
{"x" : 4 , "j" : 5 , "_id" : ObjectId("497cf87151712cf7758fbdc1")}
{"x" : 4 , "j" : 6 , "_id" : ObjectId("497cf87151712cf7758fbdc2")}
{"x" : 4 , "j" : 7 , "_id" : ObjectId("497cf87151712cf7758fbdc3")}
{"x" : 4 , "j" : 8 , "_id" : ObjectId("497cf87151712cf7758fbdc4")}
{{< /highlight >}}

请注意下, 这里循环次数是10, 但是只显示到8, 还有2条数据没有显示.

如果想继续查询下面的数据只需要使用 it 命令, 就会继续下面的数据:

{{< highlight python >}}
{"x" : 4 , "j" : 7 , "_id" : ObjectId("497cf87151712cf7758fbdc3")}
{"x" : 4 , "j" : 8 , "_id" : ObjectId("497cf87151712cf7758fbdc4")}
{{< /highlight >}}

继续

{{< highlight python >}}
> it
{"x" : 4 , "j" : 9 , "_id" : ObjectId("497cf87151712cf7758fbdc5")}
{"x" : 4 , "j" : 10 , "_id" : ObjectId("497cf87151712cf7758fbdc6")}
{{< /highlight >}}

从技术上讲 find() 返回一个游标对象. 但在上面的例子里, 并没有拿到一个游标的变量. 所以 shell 自动遍历游标, 返回一个初始化的set, 并允许我们继续用 it 迭代输出.

当然我们也可以直接用游标来输出, 不过这个是下一部分的内容了.

查询数据

在没有深入查询之前, 我们先看看怎么从一个查询中返回一个游标对象. 可以简单的通过 find() 来查询, 他返回一个任意结构的集合. 如果实现特定的查询稍后讲解.

实现上面同样的查询, 然后通过 while 来输出:

{{< highlight python >}}
> var cursor = db.things.find();
> while (cursor.hasNext()) { print(tojson(cursor.next())); }
{"name" : "mongo" , "_id" : ObjectId("497cf60751712cf7758fbdbb")}
{"x" : 3 , "_id" : ObjectId("497cf61651712cf7758fbdbc")}
{"x" : 4 , "j" : 1 , "_id" : ObjectId("497cf87151712cf7758fbdbd")}
{"x" : 4 , "j" : 2 , "_id" : ObjectId("497cf87151712cf7758fbdbe")}
{"x" : 4 , "j" : 3 , "_id" : ObjectId("497cf87151712cf7758fbdbf")}
{"x" : 4 , "j" : 4 , "_id" : ObjectId("497cf87151712cf7758fbdc0")}
{"x" : 4 , "j" : 5 , "_id" : ObjectId("497cf87151712cf7758fbdc1")}
{"x" : 4 , "j" : 6 , "_id" : ObjectId("497cf87151712cf7758fbdc2")}
{"x" : 4 , "j" : 7 , "_id" : ObjectId("497cf87151712cf7758fbdc3")}
{"x" : 4 , "j" : 8 , "_id" : ObjectId("497cf87151712cf7758fbdc4")}
{"x" : 4 , "j" : 9 , "_id" : ObjectId("497cf87151712cf7758fbdc5")}
>
{{< /highlight >}}

上面的例子显示了游标风格的迭代输出. hasNext() 函数告诉我们是否还有数据, 如果有则可以调用 next() 函数. 这里我们也用了自带的 tojson() 方法返回一个标准的 JSON 格式数据.

当我们使用的是 JavaScript shell, 可以用到JS的特性, forEach 就可以输出游标了. 下面的例子就是使用 forEach() 来循环输出:

{{< highlight python >}}
> db.things.find().forEach( function(x) { print(tojson(x));});
{"name" : "mongo" , "_id" : ObjectId("497cf60751712cf7758fbdbb")}
{"x" : 3 , "_id" : ObjectId("497cf61651712cf7758fbdbc")}
{"x" : 4 , "j" : 1 , "_id" : ObjectId("497cf87151712cf7758fbdbd")}
{"x" : 4 , "j" : 2 , "_id" : ObjectId("497cf87151712cf7758fbdbe")}
{"x" : 4 , "j" : 3 , "_id" : ObjectId("497cf87151712cf7758fbdbf")}
{"x" : 4 , "j" : 4 , "_id" : ObjectId("497cf87151712cf7758fbdc0")}
{"x" : 4 , "j" : 5 , "_id" : ObjectId("497cf87151712cf7758fbdc1")}
{"x" : 4 , "j" : 6 , "_id" : ObjectId("497cf87151712cf7758fbdc2")}
{"x" : 4 , "j" : 7 , "_id" : ObjectId("497cf87151712cf7758fbdc3")}
{"x" : 4 , "j" : 8 , "_id" : ObjectId("497cf87151712cf7758fbdc4")}
{"x" : 4 , "j" : 9 , "_id" : ObjectId("497cf87151712cf7758fbdc5")}
>
{{< /highlight >}}

forEach() 必须定义一个函数供每个游标元素调用.

在 mongo shell 里, 我们也可以把游标当作数组来用 :

{{< highlight python >}}
> var cursor = db.things.find();
> print (tojson(cursor[4]));
{"x" : 4 , "j" : 3 , "_id" : ObjectId("497cf87151712cf7758fbdbf")}
{{< /highlight >}}

使用游标时候请注意占用内存的问题, 特别是很大的游标对象, 有可能会内存溢出. 所以应该用迭代的方式来输出.

下面的示例则是把游标转换成真实的数组类型:

{{< highlight python >}}
> var arr = db.things.find().toArray();
> arr[5];
{"x" : 4 , "j" : 4 , "_id" : ObjectId("497cf87151712cf7758fbdc0")}
{{< /highlight >}}

请注意这些特性只是在 mongo shell 里使用, 而不是所有的其他应用程序驱动都支持.

MongoDB 游标对象不是没有快照 - 如果有其他用户在集合里第一次或者最后一次调用 next(), 你可以得不到游标里的数据. 所以要明确的锁定你要查询的游标.

指定条件的查询

到这里我们已经知道怎么从游标里实现一个查询并返回数据对象, 下面就来看看怎么根据指定的条件来查询.

下面的示例就是说明如何执行一个类似SQL的查询, 并演示了怎么在 MongoDB 里实现. 这是在 MongoDB shell 里查询, 当然你也可以用其他的应用驱动或者语言来实现:

{{< highlight python >}}
SELECT * FROM things WHERE name="mongo"
{{< /highlight >}}


{{< highlight python >}}
> db.things.find({name:"mongo"}).forEach(function(x) { print(tojson(x));});
{"name" : "mongo" , "_id" : ObjectId("497cf60751712cf7758fbdbb")}
>
{{< /highlight >}}


{{< highlight python >}}
SELECT * FROM things WHERE x=4
{{< /highlight >}}


{{< highlight python >}}
> db.things.find({x:4}).forEach(function(x) { print(tojson(x));});
{"x" : 4 , "j" : 1 , "_id" : ObjectId("497cf87151712cf7758fbdbd")}
{"x" : 4 , "j" : 2 , "_id" : ObjectId("497cf87151712cf7758fbdbe")}
{"x" : 4 , "j" : 3 , "_id" : ObjectId("497cf87151712cf7758fbdbf")}
{"x" : 4 , "j" : 4 , "_id" : ObjectId("497cf87151712cf7758fbdc0")}
{"x" : 4 , "j" : 5 , "_id" : ObjectId("497cf87151712cf7758fbdc1")}
{"x" : 4 , "j" : 6 , "_id" : ObjectId("497cf87151712cf7758fbdc2")}
{"x" : 4 , "j" : 7 , "_id" : ObjectId("497cf87151712cf7758fbdc3")}
{"x" : 4 , "j" : 8 , "_id" : ObjectId("497cf87151712cf7758fbdc4")}
{"x" : 4 , "j" : 9 , "_id" : ObjectId("497cf87151712cf7758fbdc5")}
>
{{< /highlight >}}

查询条件是 { a:A, b:B, ... } 类似 "where a==A and b==B and ...", 更多的查询方式可以参考 Mongo 开发教程部分.

上面显示的是所有的元素, 当然我们也可以返回特定的元素, 类似于返回表里某字段的值, 只需要在 find({x:4}) 里指定元素的名字, 比如 j:

{{< highlight python >}}
SELECT j FROM things WHERE x=4
{{< /highlight >}}


{{< highlight python >}}
> db.things.find({x:4}, {j:true}).forEach(function(x) { print(tojson(x));});
{"j" : 1 , "_id" : ObjectId("497cf87151712cf7758fbdbd")}
{"j" : 2 , "_id" : ObjectId("497cf87151712cf7758fbdbe")}
{"j" : 3 , "_id" : ObjectId("497cf87151712cf7758fbdbf")}
{"j" : 4 , "_id" : ObjectId("497cf87151712cf7758fbdc0")}
{"j" : 5 , "_id" : ObjectId("497cf87151712cf7758fbdc1")}
{"j" : 6 , "_id" : ObjectId("497cf87151712cf7758fbdc2")}
{"j" : 7 , "_id" : ObjectId("497cf87151712cf7758fbdc3")}
{"j" : 8 , "_id" : ObjectId("497cf87151712cf7758fbdc4")}
{"j" : 9 , "_id" : ObjectId("497cf87151712cf7758fbdc5")}
>
{{< /highlight >}}

请注意 "_id" 元素会一直被返回.

findOne() - 语法糖

为了方便, mongo shell (其他驱动) 避免游标的可能带来的开销, 提供一个findOne() 函数. 这个函数和 find() 参数一样, 不过他返回游标里第一条数据, 或者返回 null 空数据库.

作为一个例子, name=='mongo' 可以用很多方法来实现, 可以用 next() 来循环游标(需要校验是否为null), 或者当做数组返回第一个元素.

但是用 findOne() 方法则更简单和高效:

{{< highlight python >}}
> var mongo = db.things.findOne({name:"mongo"});
> print(tojson(mongo));
{"name" : "mongo" , "_id" : ObjectId("497cf60751712cf7758fbdbb")}
>
{{< /highlight >}}

findOne 方法更跟 find({name:"mongo"}).limit(1) 一样.

limit() 查询

你可以需要限制结果集的长度, 可以调用 limit 方法.

这是强烈推荐高性能的原因, 通过限制条数来减少网络传输, 例如:

{{< highlight python >}}
> db.things.find().limit(3);
in cursor for : DBQuery: example.things ->
{"name" : "mongo" , "_id" : ObjectId("497cf60751712cf7758fbdbb")}
{"x" : 3 , "_id" : ObjectId("497cf61651712cf7758fbdbc")}
{"x" : 4 , "j" : 1 , "_id" : ObjectId("497cf87151712cf7758fbdbd")}
>
{{< /highlight >}}

更多帮助

除非了一般的 help 之外, 你还可以查询 help 数据库和db.whatever 来查询具体的说明.

如果你对一个函数要做什么, 你可以不输入 {{()}} 这些结束的括号则可以输出实现的源码, 例如:

{{< highlight python >}}
> db.foo.insert
function (obj, _allow_dot) {
    if (!obj) {
        throw "no object passed to insert!";
    }
    if (!_allow_dot) {
        this._validateForStorage(obj);
    }
    return this._mongo.insert(this._fullName, obj);
}
{{< /highlight >}}

mongo 是一个完整的 JavaScript shell程序, 所以在 shell 里完全可以私用JS的方法、类、语法. 此外, MongoDB 定义很多自己的类和全局变量 (比如 db). 这里可以查看完整的API说明 [http://api.mongodb.org/js/](http://api.mongodb.org/js/).

接下来

看完这篇教程后下一步则看MongoDB更详细的文档 [http://www.mongodb.org/display/DOCS/Manual](http://www.mongodb.org/display/DOCS/Manual)

