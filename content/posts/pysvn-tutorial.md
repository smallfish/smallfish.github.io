---
date: 2009-11-20
title: Pysvn 程序员指南
---

这是一篇关于pysvn模块的指南.

完整和详细的API请参考 [http://pysvn.tigris.org/docs/pysvn_prog_ref.html](http://pysvn.tigris.org/docs/pysvn_prog_ref.html).

pysvn是操作Subversion版本控制的Python接口模块. 这个API接口可以管理一个工作副本, 查询档案库, 和同步两个.

该API不能创建新的仓库; 只能作用在现有仓库上. 如果你需要创建一个仓库, 请使用Subversion的svnadmin命令.

使用这个API, 你可以check out一份工作拷贝, 添加, 编辑, 和删除工作文件, 和check in, 比较, 或者放弃更改. 仓库属性, 如关键字扩展, 行字符结束, 或者忽略的列表也可以检查和控制.

Subversion 模型

Subversion是一个更新-编辑-提交的模型. 首先在本地建立一个工作副本. 在工作副本上进行修改, 最后提交到中央仓库 (可以是本地或者远程).

这个模型允许多人偶尔会同时修改同一个文件. 大多情况下. Subversion不会干预合并这些不同修改, 如果一个提交失败, 用户或者应用则要重新检查和修改然后再次提交.

常见任务

本节给出一些使用pysvn接口的常用例子. 业务可以会递归的处理目录. 添加参数recurse=False以防止这种行为; 例如, 你可以需要添加内容没有增加一个目录.

check out一份工作副本

{{< highlight python >}}
import pysvn
client = pysvn.Client()
#check out the current version of the pysvn project
client.checkout('http://localhost/example/trunk',
    './examples/pysvn')
#check out revision 11 of the pysvn project
client.checkout('http://localhost/example/trunk',
   './examples/pysvn-11',
   revision=pysvn.Revision(pysvn.opt_revision_kind.number, 11))
{{< /highlight >}}

这是一个建立example测试项目的例子，目录是examples/pysvn. 这个项目是用在剩下的例子.

添加一个文件或者目录到仓库

{{< highlight python >}}
import pysvn
# write a file foo.txt
f = file('./examples/pysvn/foo.txt', 'w')
f.write('Sample versioned file via pithon\n')
f.close()
client = pysvn.Client()
#schedule the addition;
#  the working copy will now track the file as a scheduled change
client.add('./examples/pysvn/foo.txt')
#committing the change actually adds the file to the repository
client.checkin(['./examples/pysvn/foo.txt'], 'Adding a sample file')
{{< /highlight >}}

这个例子是在工作副本中创建了'foo.txt'文件, 然后添加到仓库. 请注意Client.import_()命令会同时增加和提交. 大多数应用, 会在许多修改后再提交.

更新工作副本

{{< highlight python >}}
import pysvn
client = pysvn.Client()
client.update('./examples/pysvn')
{{< /highlight >}}

从仓库中更新其他用户修改并保存到本地副本. 大多数应用应该经常这样做以防止冲突.

提交更新到仓库

{{< highlight python >}}
import pysvn
# edit the file foo.txt
f = open('./examples/pysvn/foo.txt', 'w')
f.write('Sample versioned file via python\n')
f.close()
# checkin the change with a log message
client = pysvn.Client()
client.checkin(['./examples/pysvn'], 'Corrected spelling of python in foo.txt')
{{< /highlight >}}

提交到Subversion是原子的. 要么所有修改都成功提交, 要么提交失败. 大部分应用会提交工作副本所有修改, 如本例所示, 或者通过个别文件或者目录, 但必须是同一单位.

放弃工作副本修改

{{< highlight python >}}
import pysvn
# edit the file foo.txt
f = file('./examples/pysvn/foo.txt', 'w')
f.write('This change will never be seen\n')
f.close()
#discard the edits
client = pysvn.Client()
client.revert('./examples/pysvn/foo.txt')
{{< /highlight >}}

这丢弃在工作拷贝和恢复的文件或目录的任何未提交的未经编辑的状态变化.

正在计划增加或移除留无版本或恢复到工作拷贝.

重命名或者移动文件

{{< highlight python >}}
import pysvn
client = pysvn.Client()
#rename the file client side
client.move('./examples/pysvn/foo.txt', './examples/pysvn/foo2.txt')
#checkin the change removes the file from the repository
client.checkin(['./examples/pysvn/foo.txt', './examples/pysvn/foo2.txt'], 'Foo has become Foo2')
{{< /highlight >}}

移动或重命名文件删除旧路径或名称的文件, 并增加了在新的位置, 同时保留以前的版本有关的信息.

在这个例子里, 我们通过文件名Client.checkin()传递父目录也将是有效的.

转移和合并可以在服务器端单步完成; 可以参见仓库任务的那节例子.

从仓库中删除文件或目录

{{< highlight python >}}
import pysvn
client = pysvn.Client()
#schedule the removal;
#  the file will be removed from the working copy
client.remove('./examples/pysvn/foo2.txt')
#committing the change removes the file from the repository
client.checkin(['./examples/pysvn/foo2.txt'], 'Removing sample file')
{{< /highlight >}}

有些人把删除的文件, 或是用完全清除存储库目录. 该文件仍然存在于以前的版本, 可以通过检查或以其他方式进行审查以前修订的内容检索.

确定等待变动

{{< highlight python >}}
import pysvn
client = pysvn.Client()
changes = client.status('./examples/pysvn')
print 'files to be added:'
print [f.path for f in changes if f.text_status == pysvn.wc_status_kind.added]
print 'files to be removed:'
print [f.path for f in changes if f.text_status == pysvn.wc_status_kind.deleted]
print 'files that have changed:'
print [f.path for f in changes if f.text_status == pysvn.wc_status_kind.modified]
print 'files with merge conflicts:'
print [f.path for f in changes if f.text_status == pysvn.wc_status_kind.conflicted]
print 'unversioned files:'
print [f.path for f in changes if f.text_status == pysvn.wc_status_kind.unversioned]
{{< /highlight >}}

生成差异或补丁

{{< highlight python >}}
import pysvn
client = pysvn.Client()
diff_text = client.diff('./tmp-file-prefix-', '.')
{{< /highlight >}}

获取仓库URL

{{< highlight python >}}
import pysvn
client = pysvn.Client()
entry = client.info('.')
print 'Url:',entry.url
{{< /highlight >}}

仓库任务

本节说明任务的例子, 操纵或检查仓库.虽然共同任务, 通过本地工作副本时间的变化, 这些任务直接影响到库
获取仓库目录的清单

{{< highlight python >}}
import pysvn
client = pysvn.Client()
entry_list = client.ls('.')
{{< /highlight >}}

从仓库获取文件内容

{{< highlight python >}}
import pysvn
client = pysvn.Client()
file_content = client.cat('file.txt')
{{< /highlight >}}

创建一个标记或分支

{{< highlight python >}}
import pysvn
client = pysvn.Client()
log_message = "reason for change"
def get_log_message():
    return True, log_message
client.callback_get_log_message = get_log_message
client.copy('http://svnrepo.com/svn/trunk', 'http://svnrepo.com/svn/tag/%s' % tag_name )
{{< /highlight >}}

从仓库中转移或者重命名

{{< highlight python >}}
import pysvn
client = pysvn.Client()
client.move( 'file_old.txt', 'file_new.txt' )
{{< /highlight >}}

锁定文件

{{< highlight python >}}
import pysvn
client = pysvn.Client()
client.lock( 'file.txt', 'reason for locking' )
{{< /highlight >}}

锁定文件并锁定其他用户或者工作副本

{{< highlight python >}}
import pysvn
client = pysvn.Client()
client.lock( 'file.txt', 'reason for locking', force=True )
{{< /highlight >}}

解锁

{{< highlight python >}}
import pysvn
client = pysvn.Client()
client.unlock( 'file.txt' )
{{< /highlight >}}

解锁文件并锁定其他用户或工作副本

{{< highlight python >}}
import pysvn
client = pysvn.Client()
client.unlock( 'file.txt', force=True )
{{< /highlight >}}

测试锁定文件

Method 1:

{{< highlight python >}}
all_entries = self.client.info2( path, recurse=False )
for path, info in all_entries:
    if info['lock']:
        if info['lock']['token'] != '':
            print '%s is locked' % path
        print info['lock']['comment']
{{< /highlight >}}

Method 2:

{{< highlight python >}}
all_status = self.client.status( path, recurse=False, update=True )
for status in all_status:
    if status.entry is not None:
        if status.entry.lock_token is not None:
            print '%s is locked' % status.path
{{< /highlight >}}


