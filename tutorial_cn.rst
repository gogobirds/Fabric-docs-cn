=========
概述及教程
=========

欢迎来到Fabric!

这篇文档是对Fabric特点的快速了解之旅，也是用法的快速指导教程。
附加文档（贯穿全文）可以在 :ref:`usage documentation <usage-docs>` 中找到————请务必检查。

Fabric是什么？
===

正如“读我档案”（ ``README`` ）所述:

Fabric是一个Python库，也是命令行工具，用来简化使用SSH的应用程序部署或系统管理任务。
Fabric is a Python (2.5-2.7) library and command-line tool for streamlining the use of SSH
for application deployment or systems administration tasks.

更具体地说, Fabric是:

* 一个让你通过 **命令行** 执行 **任意Python功能** 的工具;
* 一个为了使在SSH上执行shell命令更 **简单** 和 **Python化** 的子程序库（基于底层库）。

大多数用户能轻而易举地将这两样结合起来，通过Fabric来编写和执行Python函数或任务，以达到远程服务器的自动化交互。


你好, ``fab``
===

没有“常见指引”，这也许不会是一个合适的教程::

    def hello():
        print("Hello world!")

放在你当前工作目录下叫做 ``fabfile.py`` 的Python模块文件里，
``hello`` 函数可以通过 ``fab`` 工具(Fabric安装的一部分)执行，并且符合你的预期::

    $ fab hello
    Hello world!

    Done.

这就是所有的执行。函数性使得Fabric能被当做（非常）基本构建工具使用，甚至无须引入任何API。

.. 注::

      ``fab`` 工具简单地引入了你的fabfile并执行功能或者你指示的函数。
      此处没有任何魔性方法————你能在正常Python脚本里做的都能在fabfile里实现!

.. seealso:: :ref:`execution-strategy`, :doc:`/usage/tasks`, :doc:`/usage/fab`


任务参数
====

运行时将参数传递到任务里通常会有用，正如你在常规的Python编程中所做。
Fabric通过shell-compatible符号来达到基本的支持： ``<task name>:<arg>,<kwarg>=<value>,...``.
这是不自然的，下面让我们扩展以上的实例来亲自向你问候“hello”::

    def hello(name="world"):
        print("Hello %s!" % name)

默认情况下，调用 ``fab hello`` 仍然能像之前那样运作；但是现在我们可以将它实例化::

    $ fab hello:name=Jeff
    Hello Jeff!

    Done.

那些习惯于Python编程的人可能已经猜到，这样的调用运行完全一致::

    $ fab hello:Jeff
    Hello Jeff!

    Done.

目前，你的参数值总是作为字符串出现在Python里，可能需要字符串操作等复杂类型，比如列表。
未来的版本里可能会添加强制类型转换系统以使得这样更容易。

.. seealso:: :ref:`task-arguments`

本地命令
====

正如上面所用， ``fab``          。
主要是为Fabric的API使用而设计，其中包括函数（或 **操作**），
以便于执行shell命令或文件传输，等等。
As used above, ``fab`` only really saves a couple lines of
``if __name__ == "__main__"`` boilerplate.

让我们建立一个假想的Web应用fabfile.这个示例场景如下：
 这个Web应用通过Git在远程主机``vcshost，``上被管理。
 在 ``localhost`` 上, 我们有所指Web应用的复制版本。
 当我们将改动更新到 ``vcshost`` 时，我们想要能够立即将这些改动自动地下载到远程主机 ``my_server`` 。
 我们将通过自动操作本机和远程主机的Git命令来实现。

Fabfiles通常在项目的root上工作。 usually work best at the root of a project::

    .
    |-- __init__.py
    |-- app.wsgi
    |-- fabfile.py <-- our fabfile!
    |-- manage.py
    `-- my_app
        |-- __init__.py
        |-- models.py
        |-- templates
        |   `-- index.html
        |-- tests.py
        |-- urls.py
        `-- views.py

.. note::

    我们在这里使用Django框架，但只作为一个例子————Fabric不绑定于任何外部代码库，除了SSH库以外。

对于初学者，也许我们想要在部署之前进行测试并提交到VCS::

    from fabric.api import local

    def prepare_deploy():
        local("./manage.py test my_app")
        local("git add -p && git commit")
        local("git push")

输出可能会像下面这样::

    $ fab prepare_deploy
    [localhost] run: ./manage.py test my_app
    Creating test database...
    Creating tables
    Creating indexes
    ..........................................
    ----------------------------------------------------------------------
    Ran 42 tests in 9.138s

    OK
    Destroying test database...

    [localhost] run: git add -p && git commit

    <interactive Git add / git commit edit message session>

    [localhost] run: git push

    <git push session, possibly merging conflicts interactively>

    Done.

代码本身很简单: 引入一个Fabric API函数,`~fabric.operations.local`,
并用它来和本地shell命令运行、交互.
Fabric's API的其他部分也类似————全都是python的用法.

.. seealso:: :doc:`api/core/operations`, :ref:`fabfile-discovery`


建立你的方法
======

因为Fabric是"纯Python"，你可以以任何形式自由地建立自己的fabfile.
比如，在开始任务时将它分为多个子任务::

    from fabric.api import local

    def test():
        local("./manage.py test my_app")

    def commit():
        local("git add -p && git commit")

    def push():
        local("git push")

    def prepare_deploy():
        test()
        commit()
        push()

 ``prepare_deploy`` 任务可以像之前一样被调用，但是现在, 你可以在想要的时候更细分地调用某一个子任务.


故障
==

目前我们的基本案例可以正常工作,但是如果测试失败又该怎样呢? 极大的可能是我们想设置断点，并在部署之前修复错误.

Fabric会操作检查已经执行的程序的返回值, 并且会在退出不明确的情况下中断. 我们来看看如果某个测试程序出现了错误会怎样::

    $ fab prepare_deploy
    [localhost] run: ./manage.py test my_app
    Creating test database...
    Creating tables
    Creating indexes
    .............E............................
    ======================================================================
    ERROR: testSomething (my_project.my_app.tests.MainTests)
    ----------------------------------------------------------------------
    Traceback (most recent call last):
    [...]

    ----------------------------------------------------------------------
    Ran 42 tests in 9.138s

    FAILED (errors=1)
    Destroying test database...

    Fatal error: local() encountered an error (return code 2) while executing './manage.py test my_app'

    Aborting.

哇哦!我们自己没有做任何操作: Fabric检测出了失败并且强制中断, 再也没有运行 ``commit`` 任务.

.. seealso:: :ref:`Failure handling (usage documentation) <failures>`

故障处理
----------------

但是如果我们想更灵活性地给用户一个选择呢?  :ref:`warn_only` 称为的环境设置
(或 **environment variable**[环境变量], 通常缩写为**env var**)能让你将中断操作转换为警告,
 允许存在随机应变的故障处理.

为了 ``test`` ，让我们继续看一下环境变量,之后自己动手检查一下 `~fabric.operations.local` 的结果::

    from __future__ import with_statement
    from fabric.api import local, settings, abort
    from fabric.contrib.console import confirm

    def test():
        with settings(warn_only=True):
            result = local('./manage.py test my_app', capture=True)
        if result.failed and not confirm("Tests failed. Continue anyway?"):
            abort("Aborting at user request.")

    [...]

为了介绍这个新特性，我们已经介绍了一些新的东西:

* 在Python 2.5里, ``__future__`` 的引入要求使用 ``with:`` ;
* Fabric的 `contrib.console <fabric.contrib.console>` 子模块,包含了
 `~fabric.contrib.console.confirm` 函数,用于简单的 yes/no 提示;
*  `~fabric.context_managers.settings` 上下文管理器,用于提供一块指定代码的环境设置;
* 比如 `~fabric.operations.local` 这种执行命令的操作可以返回包含结果(比如 ``.failed``,或
  ``.return_code``)信息的对象
;
* 还有 `~fabric.utils.abort` 函数,用于手动中断执行操作.

然而,尽管增添了复杂度,它仍然很容易被理解,
目前也变得更加灵活.

.. seealso:: :doc:`api/core/context_managers`, :ref:`env-vars`


建立连接
====

让我们通过by putting in the keystone开始封装fabfile : 一个 ``deploy``
任务的目的是运行在一个或多个远程服务器上,并且确保代码是最新的::

    def deploy():
        code_dir = '/srv/django/myproject'
        with cd(code_dir):
            run("git pull")
            run("touch app.wsgi")

在这里,我们又引入了一些新的概念:

* Fabric就是Python -- 所以我们可以自由使用常用的Python代码设计,比如便令和字符串插入;
* `~fabric.context_managers.cd`,通过 ``cd
  /to/some/directory`` 调用的前缀命令的简单方式.这个和  `~fabric.context_managers.lcd`
  相似,在本地也做了同样的事.
* `~fabric.operations.run` 和 `~fabric.operations.local` 相似，但是**远程**运行而不是在本地.

我们同样需要确认新函数是在文件顶部引入的::

    from __future__ import with_statement
    from fabric.api import local, settings, abort, run, cd
    from fabric.contrib.console import confirm

有了这些变动,我们开始部署::

    $ fab deploy
    No hosts found. Please specify (single) host string for connection: my_server
    [my_server] run: git pull
    [my_server] out: Already up-to-date.
    [my_server] out:
    [my_server] run: touch app.wsgi

    Done.

我们从未在fabfile上指定任何连接信息,所以Fabric不知道应该在哪台主机执行远程命令.
当出现这种情况时,
Fabric会在运行时提示我们.连接定义使用了类似SSH的"主机字符串" (e.g. ``user@host:port``)
并且将使用你本地的用户名作为默认值 -- 所以在这个例子里,我们仅仅需要制定主机名, ``my_server``.


远程交互
----

如果你已经检查过源代码,``git pull`` 将运作良好 --
但如果是第一次部署又会怎样呢? 处理这种情况和做初始化 ``git clone``也同样令人乐意::

    def deploy():
        code_dir = '/srv/django/myproject'
        with settings(warn_only=True):
            if run("test -d %s" % code_dir).failed:
                run("git clone user@vcshost:/path/to/repo/.git %s" % code_dir)
        with cd(code_dir):
            run("git pull")
            run("touch app.wsgi")

正如之前对 `~fabric.operations.local` 的调用, `~fabric.operations.run`
同样让我们建立了基于可执行的shell命令的明确的Python-level逻辑. 然而,这有趣的部分就是 ``git clone`` 调用:
既然我们使用了Git的SSH访问服务器上存储库的方法,这就意味着我们的远程 `~fabric.operations.run` 调用
将需要验证自己本身.

以前的Fabric版本里(和类似的高级别SSH库)在limbo里运行远程程序, 不能在本地终端运行.
当你非常需要输入密码或者和其他远程程序的交互时,这就是问题所在.

Fabric 1.0 和之后的版本都打破了这个限制并且确保你可以一直和另一端交流.
我们来看看，运行更新之后在新服务器上不经Git检查的 ``deploy`` 任务时会发生什么::

    $ fab deploy
    No hosts found. Please specify (single) host string for connection: my_server
    [my_server] run: test -d /srv/django/myproject

    Warning: run() encountered an error (return code 1) while executing 'test -d /srv/django/myproject'

    [my_server] run: git clone user@vcshost:/path/to/repo/.git /srv/django/myproject
    [my_server] out: Cloning into /srv/django/myproject...
    [my_server] out: Password: <enter password>
    [my_server] out: remote: Counting objects: 6698, done.
    [my_server] out: remote: Compressing objects: 100% (2237/2237), done.
    [my_server] out: remote: Total 6698 (delta 4633), reused 6414 (delta 4412)
    [my_server] out: Receiving objects: 100% (6698/6698), 1.28 MiB, done.
    [my_server] out: Resolving deltas: 100% (4633/4633), done.
    [my_server] out:
    [my_server] run: git pull
    [my_server] out: Already up-to-date.
    [my_server] out:
    [my_server] run: touch app.wsgi

    Done.

注意 ``Password:`` 提示 -- 这是在我们的Web服务器上的远程 ``git`` 调用,要求Git服务器的密码.
我们可以输入并且复制也一样能正常继续.

.. seealso:: :doc:`/usage/interactivity`


.. _defining-connections:

预先定义的连接
-------

在运行时指定连接信息真的会跑得很快,所以Fabric提供了在fabfile或者命令行里解决的一些办法.
我们不会在这里提到全部的方法,但是我们将会展示给你最常用的一种: 设置全局主机列表, :ref:`env.hosts <hosts>`.

:doc:`env <usage/env>` 是一个全局的字典型的对象,控制Fabric的很多设置,
并且也可以写入属性(实际上,`~fabric.context_managers.settings`,如上所见,只是一个封装器.)
从而我们可以在fabfile的模块级别的顶部上修改，比如这样::

    from __future__ import with_statement
    from fabric.api import *
    from fabric.contrib.console import confirm

    env.hosts = ['my_server']

    def test():
        do_test_stuff()

当 ``fab`` 加载我们的fabfile时,我们对 ``env`` 的修改将会执行,
保存了我们的设置改动. 最终的结果如上: ``deploy``
任务将针对 ``my_server`` 服务器运行.

This is also how you can tell Fabric to run on multiple remote systems at once:
because ``env.hosts`` is a list, ``fab`` iterates over it, calling the given
task once for each connection.

.. seealso:: :doc:`usage/env`, :ref:`host-lists`


Conclusion
==========

Our completed fabfile is still pretty short, as such things go. Here it is in
its entirety::

    from __future__ import with_statement
    from fabric.api import *
    from fabric.contrib.console import confirm

    env.hosts = ['my_server']

    def test():
        with settings(warn_only=True):
            result = local('./manage.py test my_app', capture=True)
        if result.failed and not confirm("Tests failed. Continue anyway?"):
            abort("Aborting at user request.")

    def commit():
        local("git add -p && git commit")

    def push():
        local("git push")

    def prepare_deploy():
        test()
        commit()
        push()

    def deploy():
        code_dir = '/srv/django/myproject'
        with settings(warn_only=True):
            if run("test -d %s" % code_dir).failed:
                run("git clone user@vcshost:/path/to/repo/.git %s" % code_dir)
        with cd(code_dir):
            run("git pull")
            run("touch app.wsgi")

This fabfile makes use of a large portion of Fabric's feature set:

* defining fabfile tasks and running them with :doc:`fab <usage/fab>`;
* calling local shell commands with `~fabric.operations.local`;
* modifying env vars with `~fabric.context_managers.settings`;
* handling command failures, prompting the user, and manually aborting;
* and defining host lists and `~fabric.operations.run`-ning remote commands.

However, there's still a lot more we haven't covered here! Please make sure you
follow the various "see also" links, and check out the documentation table of
contents on :doc:`the main index page <index>`.

Thanks for reading!
