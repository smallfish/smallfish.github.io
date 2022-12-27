---
date: 2010-02-06
title: PostgreSQL RPM 安装笔记

tags:
- PostgreSQL

---

测试环境：REHL 5.3
PostgreSQL版本：8.4.2

1. 首先检查下是否已经有PostgreSQL安装程序(俺的机器有pg-libs 8.1，无视之)

{{< highlight python >}}

shell> rpm -qa | grep postgres

{{< /highlight >}}

2. 下载最新的8.4.2RPM安装包，这个FTP速度挺快的。:)

{{< highlight python >}}

shell> wget http://ftp.easynet.be/postgresql/binary/v8.4.2/linux/rpms/redhat/rhel-5-x86_64/postgresql-server-8.4.2-1PGDG.rhel5.x86_64.rpm
shell> wget http://ftp.easynet.be/postgresql/binary/v8.4.2/linux/rpms/redhat/rhel-5-x86_64/postgresql-contrib-8.4.2-1PGDG.rhel5.x86_64.rpm
shell> wget http://ftp.easynet.be/postgresql/binary/v8.4.2/linux/rpms/redhat/rhel-5-x86_64/postgresql-libs-8.4.2-1PGDG.rhel5.x86_64.rpm
shell> wget http://ftp.easynet.be/postgresql/binary/v8.4.2/linux/rpms/redhat/rhel-5-x86_64/postgresql-devel-8.4.2-1PGDG.rhel5.x86_64.rpm
shell> wget http://ftp.easynet.be/postgresql/binary/v8.4.2/linux/rpms/redhat/rhel-5-x86_64/postgresql-8.4.2-1PGDG.rhel5.x86_64.rpm
shell> wget http://ftp.easynet.be/postgresql/binary/v8.4.2/linux/rpms/redhat/rhel-5-x86_64/postgresql-plpython-8.4.2-1PGDG.rhel5.x86_64.rpm

{{< /highlight >}}

3. 安装PostgreSQL(要注意下顺序)，首先需要更新pg-libs版本。
后面几个不需要的话可以不装。主要是一些扩展功能。

{{< highlight python >}}

shell> rpm -ivh postgresql-libs-8.4.2-1PGDG.rhel5.x86_64.rpm
shell> rpm -ivh postgresql-8.4.2-1PGDG.rhel5.x86_64.rpm
shell> rpm -ivh postgresql-server-8.4.2-1PGDG.rhel5.x86_64.rpm
shell> rpm -ivh postgresql-contrib-8.4.2-1PGDG.rhel5.x86_64.rpm
shell> rpm -ivh postgresql-devel-8.4.2-1PGDG.rhel5.x86_64.rpm
shell> rpm -ivh postgresql-plpython-8.4.2-1PGDG.rhel5.x86_64.rpm

{{< /highlight >}}

4. RPM安装完后，需要初始化PostgreSQL库。service初次启动会提示。
如果是源码安装这个过程就是对应的initdb -D，指定data目录。RPM默认对应目录是/var/lib/pgsql/data。

{{< highlight python >}}

shell> service postgresql initdb

{{< /highlight >}}

5. service启动PostgreSQL

{{< highlight python >}}

shell> service postgresql start

{{< /highlight >}}


到上面这一步基本是安装完成了。下面的是修改数据库用户密码和登陆相关。

6. 切换到postgres用户，修改数据库密码。(注意系统用户和数据库用户密码是两个概念，虽然名字都叫postgres)
修改完需要重启数据库，这里咱不重启，等修改完认证配置再一起重启。

{{< highlight python >}}

shell> su - postgres
shell> psql
postgres=# ALTER USER postgres WITH PASSWORD '123456';
postgres=# \q

{{< /highlight >}}

7. 修改认证文件/var/lib/pgsql/data/pg_hba.conf，登陆使用密码。md5格式。

{{< highlight python >}}

shell> vi /var/lib/pgsql/data/pg_hba.conf
修改ident为md5 (local, host)

{{< /highlight >}}

8. service重启PostgreSQL

{{< highlight python >}}

shell> service postgresql restart

{{< /highlight >}}

9. 再次进入测试，应该会提示输入密码鸟 :)

{{< highlight python >}}

shell>psql -U postgres

{{< /highlight >}}


