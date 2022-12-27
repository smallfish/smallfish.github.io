---
date: 2010-07-01
title: Pylons 入门实例教程 – 数据库操作
---

前面两篇入门，讲述了 Pylons 大致开发的流程、表单以及文件上传，思路大致跟传统的开发类似。本篇简单讲述下在 Pylons 如何使用数据库。

本篇侧重点是使用 ORM 框架 [http://www.sqlalchemy.org/](http://www.sqlalchemy.org/)。其实本人最早是研究了一下 Storm，后来听虾哥（@marchliu）在应用里不是很爽之，遂关注了下他推荐的 SQLAlchemy。当然，你也可以对应数据库的 DB-API 库来进行操作。

示例代码的数据库是 [http://www.postgresql.org](http://www.postgresql.org)。至于 Pg 配置和使用这里不再累赘，请狗之。

Debian/Ubuntu 安装很简单：

{{< highlight python >}}
sudo aptitude install python-psycopg2
{{< /highlight >}}

建立一个测试数据库，比如 test：

{{< highlight python >}}
smallfish@debian:~/workspace/python/hello$ su postgres
postgres@debian:/home/smallfish/workspace/python/hello$ createdb -O smallfish test
postgres@debian:/home/smallfish/workspace/python/hello$ exit
smallfish@debian:~/workspace/python/hello$ psql -h 127.0.0.1 -p 5432 -U smallfish test
用户 smallfish 的口令：
psql (8.4.4)
SSL连接 (加密：DHE-RSA-AES256-SHA，位元：256)

输入 "help" 来获取帮助信息.

test=#
{{< /highlight >}}

数据库的部分已经OK，下面就是来倒腾 Pylons 啦。

建立新项目，加入支持数据库部分，注意 Enter sqlalchemy那个选项，默认是 False，改成 True：

{{< highlight python >}}
smallfish@debian:~/workspace/python$ paster create -t pylons hellodb
Selected and implied templates:
  Pylons#pylons  Pylons application template

Variables:
  egg:      hellodb
  package:  hellodb
  project:  hellodb
Enter template_engine (mako/genshi/jinja2/etc: Template language) ['mako']:
Enter sqlalchemy (True/False: Include SQLAlchemy 0.5 configuration) [False]: True
Creating template pylons
Creating directory ./hellodb
{{< /highlight >}}

改成 True 之后在自动生成的 development.ini 里就有对应的数据库配置选项了。

再建立新的 db controller：

{{< highlight python >}}
smallfish@debian:~/workspace/python$ cd hellodb/
smallfish@debian:~/workspace/python/hellodb$ paster controller db
Creating /home/smallfish/workspace/python/hellodb/hellodb/controllers/db.py
Creating /home/smallfish/workspace/python/hellodb/hellodb/tests/functional/test_db.py
{{< /highlight >}}

编辑 development.ini，添加数据库配置部分。smallfish:123456 是对应的 PostgreSQL 用户名/密码，127.0.0.1:5432 是对应的主机地址/端口号，最后的test则是数据库名。

{{< highlight python >}}
# SQLAlchemy database URL
sqlalchemy.url = postgresql://smallfish:123456@127.0.0.1:5432/test
{{< /highlight >}}

编辑 hellodb/model/__init__.py，加上一个叫 msg 的表和字段的定义：

{{< highlight python >}}
"""The application's model objects"""
from hellodb.model.meta import Session, metadata

from sqlalchemy import orm, schema, types
from datetime import datetime

def init_model(engine):
    """Call me before using any of the tables or classes in the model"""
    Session.configure(bind=engine)

def now():
    return datetime.now()

msg_table = schema.Table('msg', metadata,
    schema.Column('id', types.Integer, schema.Sequence('msg_seq_id', optional=True), primary_key=True),
    schema.Column('content', types.Text(), nullable=False),
    schema.Column('addtime', types.DateTime(), default=now),
)

class Msg(object):
    pass

orm.mapper(Msg, msg_table)
{{< /highlight >}}

示例 Msg 表很简单，三个字段：ID、内容和时间。

上面的代码除去导入 sqlclchemy 包里几个库，基本上有一个对应表的字段定义，还有一个空的 Msg 对象。

最后一行，则是做一个 map 的动作，把 Msg 映射到 msg_table 上。

下面是不是要在数据库里建立对应的表呢？有个简单的办法 可以初始化数据库：paster setup-app development.ini：

{{< highlight python >}}
smallfish@debian:~/workspace/python/hellodb$ paster setup-app development.ini
Running setup_config() from hellodb.websetup
20:08:43,619 INFO  [sqlalchemy.engine.base.Engine.0x...854c] [MainThread] select version()
20:08:43,619 INFO  [sqlalchemy.engine.base.Engine.0x...854c] [MainThread] {}
20:08:43,625 INFO  [sqlalchemy.engine.base.Engine.0x...854c] [MainThread] select current_schema()
20:08:43,625 INFO  [sqlalchemy.engine.base.Engine.0x...854c] [MainThread] {}
20:08:43,631 INFO  [sqlalchemy.engine.base.Engine.0x...854c] [MainThread] select relname from pg_class c join pg_namespace n on n.oid=c.relnamespace where n.nspname=current_schema() and lower(relname)=%(name)s
20:08:43,631 INFO  [sqlalchemy.engine.base.Engine.0x...854c] [MainThread] {'name': u'msg'}
20:08:43,637 INFO  [sqlalchemy.engine.base.Engine.0x...854c] [MainThread]
CREATE TABLE msg (
        id SERIAL NOT NULL,
        content TEXT NOT NULL,
        addtime TIMESTAMP WITHOUT TIME ZONE,
        PRIMARY KEY (id)
)

20:08:43,637 INFO  [sqlalchemy.engine.base.Engine.0x...854c] [MainThread] {}
20:08:43,732 INFO  [sqlalchemy.engine.base.Engine.0x...854c] [MainThread] COMMIT
{{< /highlight >}}

可以看到上面的输出日志，包括了建表的SQL语句。其中 SERIAL 对应上面 __init__.py 里 Column 的 Seq 定义。serial 类型 在 PostgreSQL 可以看成类似 MySQL 的自增ID（auto_increment） 。

现在进入 PostgreSQL 查询数据库，就可以看到表以及序列已经建立。

{{< highlight python >}}
test=# \d
                  关联列表
 架构模式 |    名称    |  型别  |  拥有者
----------+------------+--------+-----------
 public   | msg        | 资料表 | smallfish
 public   | msg_id_seq | 序列数 | smallfish
(2 行记录)
{{< /highlight >}}

到这里，准备工作已经完毕，包括了初始化数据库，配置文件还有示例 controller。

下面就在 controller 代码里增加读写数据库的功能吧。

首先建立一个表单模板 db.htm ，用来添加并保存到数据库表中：

{{< highlight python >}}
<form action="/db/add" method="post">
    <input type="text" name="content" />
    <br />
    <input type="submit" value="save" />
</form>
{{< /highlight >}}

对应 controller index 修改成，很简单。返回到模板：

{{< highlight python >}}
class DbController(BaseController):

    def index(self):
        return render('/db.htm')
{{< /highlight >}}

添加 add 方法，对应上面 form 中的 /db/add 路径：

{{< highlight python >}}
    def add(self):
        content = request.POST['content']
        from hellodb import model
        msg = model.Msg()
        msg.content = content
        model.meta.Session.add(msg)
        model.meta.Session.commit()
        return "add %s ok..." % content
{{< /highlight >}}

添加部分简单完成。获取 POST 文本框，然后初始化一个 Msg 对象（上面 model 里定义的）。

注意 add 之后，必须手动 commit，这样才会真正保存到数据库。

浏览器访问一下：http://127.0.0.1:5000/db/index，随意添加一点数据吧，这个时候你可以在 PostgreSQL 里查询已经数据已经加进来了。

下面在 index 方法传递一些值到模板，输出刚才已经添加的数据：

{{< highlight python >}}
    def index(self):
        from hellodb import model
        c.msgs = model.meta.Session.query(model.Msg).all()
        return render('/db.htm')
{{< /highlight >}}

c.msgs 可以理解成全局变量，关于 c 的定义在 controller前几行就应该看到了。修改模板 db.htm 显示记录：

{{< highlight python >}}
% for msg in c.msgs:
<p>${msg.id}: ${msg.content} / ${msg.addtime}</p>
% endfor
{{< /highlight >}}

很简单，只是一个普通 for 循环，遍历 index 方式里传递的 c.msgs。Mako模板还是很易读的吧？

继续刷新下：http://127.0.0.1:5000/db/index，可以在页面上看到已经添加的数据了。

在狂输入了几十条之后，在一页里显示是不是忒土鳖了？

下面再介绍下 Pylons 里 webhelper 里一个分页组件的用法，当然你也可以自己写分页算法。下面是示例：

{{< highlight python >}}
    def list(self):
        from webhelpers import paginate
        from hellodb import model
        msgs = model.meta.Session.query(model.Msg)
        c.paginator = paginate.Page(
            msgs,
            page=int(request.params.get('page', 1)),
            items_per_page = 10,
        )
        return render("/list.htm")
{{< /highlight >}}

导入 paginate，然后把查询的数据库对象当参数传递给 paginate.Page，里面的page则是页面的传递的页数参数，items_per_page 就好理解多了，就是多少条一页。这里是10条。

对应的模板 list.htm 如下：

{{< highlight python >}}
<pre>
% if len(c.paginator):

% for msg in c.paginator:
<p>${msg.id}: ${msg.content}</p>
% endfor

<p> ${c.paginator.pager("count: $item_count $first_item to $last_item , $link_first $link_previous $link_next $link_last")} </p>
% endif
{{< /highlight >}}

for 部分如同上面示例，下面加了一行pager。里面一些变量从名字上就可以看出功能了。包括了总条数、当前是第几到第几条，然后就是常用的首页、上页、下页和最后一页。

这里链接的文字都是<<， <， >， >>。想改成文字请查看文档吧。。如果是第一页，是不会显示首页和上一页的。这个做过分页的一般都写过类似的代码吧。

现在访问：http://127.0.0.1:5000/db/list，想看到效果当然你得多填点数据哦。10条才会显示分页的挖。

好了，这里对数据库增加和显示部分都已经有示例代码了，当然最后还有一个分页用法。至于删除和更新之类请参考 SQLAlchemy 文档吧。

