====
执行模型
====

如果你已经读过 :doc:`../tutorial`, 你应该熟悉对Fabric的基本使用(单主机上执行单任务) 然而,
在很多情况下，你会发现需要在多个主机上执行多个任务. 也许你想把一个大任务分割为小的部分,
或者删除一组服务器上的老用户. 这种特定规则的情景下如何执行任务的.

本文档将探讨Fabric的执行模型，包括主执行循环，如何定义主机列表，如何连接等.


.. _execution-strategy:

执行策略
====

Fabric 默认采用串行执行单任务的方式, 虽然在Fabric 1.3中可以采用并行模式来替代
(参见 :doc:`/usage/parallel`). 默认行为如下：

* 创建一个任务列表，简单地按照列表顺序作为 :doc:`fab <fab>` 的参数即可
* 对于每一个任务，从各种来源生成一个特定任务主机列表。(参见 :ref:`host-lists` 的更多细节.)
* 任务列表按照顺序执行，每个任务只执行一次在主机列表内的每个主机.
* 在主机列表没有主机上的被认为是本地任务，只会运行一次。

因此, 给出一个 fabfile::

    from fabric.api import run, env

    env.hosts = ['host1', 'host2']

    def taskA():
        run('ls')

    def taskB():
        run('whoami')

通过以下命令调用::

    $ fab taskA taskB

可以看到以下输出:

* ``taskA`` executed on ``host1``
* ``taskA`` executed on ``host2``
* ``taskB`` executed on ``host1``
* ``taskB`` executed on ``host2``

虽然这种方式很简单, 能够将任务简单的组合到一起（不像发送任务到多个主机分别调用的工具）同时可以使用脚本一样的逻辑，
从输出得到反馈或得到命令的返回码，从而决定下一步做什么


定义任务
====

关于Fabric task的构成和组成的细节，参见:doc:`/usage/tasks`.

定义主机列表
======

除非你只是将Fabric作为一个简单执行任务构建的系统（可以但不是主要用法）是没有好处的，
不指定特定的主机去执行。有很多方法去实现，从全局到每个任务都可以根据需要混合和匹配。

.. _host-strings:

主机
----

主机，在这种上下文中通常也被称为"主机字符串": 一个由用户名，主机名和端口组合而成的Python
字符串，如``username@hostname:port``这种形式，用户和端口可以被省略（由``@`` 或 ``:``
所关联），由本地用户名和默认端口22来代替。因此``admin@foo.com:222``, ``deploy@website``
和 ``nameserver1`` 都是有效的主机串。

同时也支持IPv6, 比如 ``::1``, ``[::1]:1222``, ``user@2001:db8::1`` or
``user@[2001:db8::1]:1222``. 方括号作为地址与端口的分割是必要的。如果端口号不需要，方括号是可选的
如果主机串由命令行参数执行，在一些shell中可能需要转义方括号。

.. 注意::
    用户和主机通过最后一个``@``符号分隔，所以email作为用户名是有效的会被正确解析。

执行期间，Fabric格式化主机串并储存每个部分（用户名/主机名/端口号）到环境字典中，如果需要使用或任务引用。
参见:doc:`env`.

.. _execution-roles:

角色
----

主机串对应单个主机，有时候安排主机到组很有用。也许你需要一些Web Server 在负载均衡
或者想要在“所有client服务器”执行一个任务。角色提供一种定义字符串对应到主机串列表的功能.
用来替代每次写出主机列表.

该映射通过一个字典来定义, ``env.roledefs``, 为了使用它必须通过修改fabfile
简单的例子::

    from fabric.api import env

    env.roledefs['webservers'] = ['www1', 'www2', 'www3']

由于 ``env.roledefs`` 默认为空, 又可以重新定义它而不必担心丢失任何信息
(前提当然是你不加载其他fabfiles去修改它)::

    from fabric.api import env

    env.roledefs = {
        'web': ['www1', 'www2', 'www3'],
        'dns': ['ns1', 'ns2']
    }

角色的定义不仅是主机的必要配置，还可以选择主机执行特定配置 .
通过定义角色字典和和在``hosts``下定义主机串实现::

    from fabric.api import env

    env.roledefs = {
        'web': {
            'hosts': ['www1', 'www2', 'www3'],
            'foo': 'bar'
        },
        'dns': {
            'hosts': ['ns1', 'ns2'],
            'foo': 'baz'
        }
    }

除了列表/可迭代类型，该值``env.roledefs``（或字典样式定义``hosts``键的值）是可调用，
当任务运行时被调用而不是模块加载时.(例如，你可以在连接远程服务器时获取角色定义，
而不必担心在调用``fab --list``是时加载fabfile引起延迟)

以任何方式使用角色都不是必须的 -- 它仅仅是提供了一个便利的方式在你有通用的服务器分组情况下

.. 版本变更:: 0.9.2
    增加可调用的值``roledefs``.

.. _host-lists:

如何构造主机列表
--------

有很多种办法指定主机列表，无论是全局或者按任务，通常这些方法会被互相覆盖而不是合并到一起
（虽然有可能在未来的版本变更）每种方式都被分为两部分，用于主机和用于角色。

全局变量, 通过 ``env``
~~~

修改环境字典的键值对是设定主机或角色最通用的方式，:doc:`env <env>`: ``hosts`` and ``roles``.
这些变量的值在运行时被检查，从而构成每个任务的主机的列表。

因此，通过fabfile被导入时会被设定为模块级别的变量::

    from fabric.api import env, run

    env.hosts = ['host1', 'host2']

    def mytask():
        run('ls /var/www')

像这样的fabfile，通过``fab mytask``运行，将会依次在``host1``，``host2``上执行``mytask``.

由于*每个* 任务都会检查环境变量，意味着可以根据需要在一个任务中修改``env`` 将会影响到后续任务::

    from fabric.api import env, run

    def set_hosts():
        env.hosts = ['host1', 'host2']

    def mytask():
        run('ls /var/www')

运行 ``fab set_hosts mytask``, ``set_hosts`` 是一个"本地"任务 -- 它的主机列表为空
-- 但 ``mytask`` 会在定义两个主机后运行.

.. note::

    这种技术通常用来创建虚拟"角色".在角色完全实现的情况下显得没必要，但有时候会显得很有用

如果 ``env.hosts`` is ``env.roles`` (不要与``env.roledefs``混淆!) 被给定，可以在``env.roledefs``
中查找作为角色名列表.

全局变量, 通过命令行参数
~~~

除了在模块级别修改 ``env.hosts``, ``env.roles`` 和 ``env.exclude_hosts`` ,
也可以通过定义逗号分隔的命令行选项 :option:`--hosts/-H <-H>` and :option:`--roles/-R <-R>`
例如.::

    $ fab -H host1,host2 mytask

这种调用相当于 ``env.hosts = ['host1', 'host2']`` -- 参数解析器在解析时会寻找这些参数并修改``env``

.. note::

    事实上，使用这些选项可能会设定成单个主机或角色，Fabric在得到字符串时调用的是``string.split(',')``，
    所以没有逗号的字符串会变成单个列表.

明白命令行选项在你的fabfile加载前被解释时重要的，在fabfile重新定义``env.hosts`` or
``env.roles``会覆盖它们.

如果希望命令行定义和fabfile定义的主机列表无损合并，确保在fabfile中使用``env.hosts.extend()``::

    from fabric.api import env, run

    env.hosts.extend(['host3', 'host4'])

    def mytask():
        run('ls /var/www')

当这个fabfile通过 ``fab -H host1,host2 mytask``运行时, 在``mytask``执行时，
``env.hosts`` 将会包含``['host1', 'host2', 'host3', 'host4']``.

.. note::

    ``env.hosts`` 就是一个Python列表对象 -- 所以可以使用``env.hosts.append()``
    或者其他你想用的列表方法.

.. _hosts-per-task-cli:

单任务，通过命令行参数
~~~

设置全局主机列表足以让所有任务跑在相同的主机上.但有时是不需要这样的，所以Fabric提供一些方法
更精确和特殊的指定单个任务的主机列表。第一个方法是指定任务参数.

如:doc:`fab` 所述, 可以通过特定的命令行语法指定任务参数. 除了命名任务的实际参数，还可以设定
``host``, ``hosts``, ``role`` or ``roles`` "arguments" 在Fabric建立主机列表时被解析
(在传递到任务时被删除.)

.. note::

    由于逗号已经习惯用来分割任务参数，而分号用来划分``hosts`` or ``roles`` 主机和角色.
    此外，必须加上引号以方式shell解析分号.

运行下面的fabfile, 除了没有定义主机信息和使用过的一样::

    from fabric.api import run

    def mytask():
        run('ls /var/www')

为``mytask``指定特定的主机, 如下执行::

    $ fab mytask:hosts="host1;host2"

它将覆盖所有的主机列表确保``mytask``仅仅在两个主机上执行.

单任务, 通过装饰器
~~~

如果一个任务总是运行在一个预定义的主机列表，你可能希望在fabfile中指定.
可通过`~fabric.decorators.hosts` 或 `~fabric.decorators.roles`装饰任务函数
装饰器需要一个可变的参数列表，例如::

    from fabric.api import hosts, run

    @hosts('host1', 'host2')
    def mytask():
        run('ls /var/www')

也可以传入一个可迭代的参数 如::

    my_hosts = ('host1', 'host2')
    @hosts(my_hosts)
    def mytask():
        # ...

在使用时，这些装饰器覆盖了``env``对特定主机列表的检查 (尽管``env`` 没有被任何方法修改，只是简单地被忽略.)
因此，即使fabfile定义了 ``env.hosts``或者通过选项 :option:`--hosts/-H <-H>` 调用 :doc:`fab <fab>`
``mytask``仍然只会在``['host1', 'host2']``上执行.

然而,装饰主机列表**不会**覆盖通过命令行指定的任务，前一节给出了解释.

优先级
~~~

我们已经一起研究了设定主机列表的方法，然而，为了更清晰，快速回顾一下:

* 单任务，通过命令行(``fab mytask:host=host1``)，可覆盖其他所有方法
* 单任务，通过装饰特定主机列表(``@hosts('host1')``)，可覆盖``env`` 变量.
* 在fabfile中全局指定主机列表(``env.hosts = ['host1']``)*可以* 覆盖通过命令行指定的列表
  但只会在你无意中（或希望）的情况下.
* 在命令行中全局指定主机列表(``--hosts=host1``)，将会初始化``env``变量，仅此而已.

这个逻辑顺序可能在未来版本中变得更加一致 (例如，使用选项 :option:`--hosts <-H>`在某种程度上
优先于``env.hosts``在同样使用命令行指定单任务）但只会在向后兼容的版本中出现.

.. _combining-host-lists:

Combining host lists
--------------------

There is no "unionizing" of hosts between the various sources mentioned in
:ref:`host-lists`. If ``env.hosts`` is set to ``['host1', 'host2', 'host3']``,
and a per-function (e.g.  via `~fabric.decorators.hosts`) host list is set to
just ``['host2', 'host3']``, that function will **not** execute on ``host1``,
because the per-task decorator host list takes precedence.

However, for each given source, if both roles **and** hosts are specified, they
will be merged together into a single host list. Take, for example, this
fabfile where both of the decorators are used::

    from fabric.api import env, hosts, roles, run

    env.roledefs = {'role1': ['b', 'c']}

    @hosts('a', 'b')
    @roles('role1')
    def mytask():
        run('ls /var/www')

Assuming no command-line hosts or roles are given when ``mytask`` is executed,
this fabfile will call ``mytask`` on a host list of ``['a', 'b', 'c']`` -- the
union of ``role1`` and the contents of the `~fabric.decorators.hosts` call.


.. _deduplication:

Host list deduplication
-----------------------

By default, to support :ref:`combining-host-lists`, Fabric deduplicates the
final host list so any given host string is only present once. However, this
prevents explicit/intentional running of a task multiple times on the same
target host, which is sometimes useful.

To turn off deduplication, set :ref:`env.dedupe_hosts <dedupe_hosts>` to
``False``.


.. _excluding-hosts:

Excluding specific hosts
------------------------

At times, it is useful to exclude one or more specific hosts, e.g. to override
a few bad or otherwise undesirable hosts which are pulled in from a role or an
autogenerated host list.

.. note::
    As of Fabric 1.4, you may wish to use :ref:`skip-bad-hosts` instead, which
    automatically skips over any unreachable hosts.

Host exclusion may be accomplished globally with :option:`--exclude-hosts/-x
<-x>`::

    $ fab -R myrole -x host2,host5 mytask

If ``myrole`` was defined as ``['host1', 'host2', ..., 'host15']``, the above
invocation would run with an effective host list of ``['host1', 'host3',
'host4', 'host6', ..., 'host15']``.

    .. note::
        Using this option does not modify ``env.hosts`` -- it only causes the
        main execution loop to skip the requested hosts.

Exclusions may be specified per-task by using an extra ``exclude_hosts`` kwarg,
which is implemented similarly to the abovementioned ``hosts`` and ``roles``
per-task kwargs, in that it is stripped from the actual task invocation. This
example would have the same result as the global exclude above::

    $ fab mytask:roles=myrole,exclude_hosts="host2;host5"

Note that the host list is semicolon-separated, just as with the ``hosts``
per-task argument.

Combining exclusions
~~~~~~~~~~~~~~~~~~~~

Host exclusion lists, like host lists themselves, are not merged together
across the different "levels" they can be declared in. For example, a global
``-x`` option will not affect a per-task host list set with a decorator or
keyword argument, nor will per-task ``exclude_hosts`` keyword arguments affect
a global ``-H`` list.

There is one minor exception to this rule, namely that CLI-level keyword
arguments (``mytask:exclude_hosts=x,y``) **will** be taken into account when
examining host lists set via ``@hosts`` or ``@roles``. Thus a task function
decorated with ``@hosts('host1', 'host2')`` executed as ``fab
taskname:exclude_hosts=host2`` will only run on ``host1``.

As with the host list merging, this functionality is currently limited (partly
to keep the implementation simple) and may be expanded in future releases.


.. _execute:

Intelligently executing tasks with ``execute``
==============================================

.. versionadded:: 1.3

Most of the information here involves "top level" tasks executed via :doc:`fab
<fab>`, such as the first example where we called ``fab taskA taskB``.
However, it's often convenient to wrap up multi-task invocations like this into
their own, "meta" tasks.

Prior to Fabric 1.3, this had to be done by hand, as outlined in
:doc:`/usage/library`. Fabric's design eschews magical behavior, so simply
*calling* a task function does **not** take into account decorators such as
`~fabric.decorators.roles`.

New in Fabric 1.3 is the `~fabric.tasks.execute` helper function, which takes a
task object or name as its first argument. Using it is effectively the same as
calling the given task from the command line: all the rules given above in
:ref:`host-lists` apply. (The ``hosts`` and ``roles`` keyword arguments to
`~fabric.tasks.execute` are analogous to :ref:`CLI per-task arguments
<hosts-per-task-cli>`, including how they override all other host/role-setting
methods.)

As an example, here's a fabfile defining two stand-alone tasks for deploying a
Web application::

    from fabric.api import run, roles

    env.roledefs = {
        'db': ['db1', 'db2'],
        'web': ['web1', 'web2', 'web3'],
    }

    @roles('db')
    def migrate():
        # Database stuff here.
        pass

    @roles('web')
    def update():
        # Code updates here.
        pass

In Fabric <=1.2, the only way to ensure that ``migrate`` runs on the DB servers
and that ``update`` runs on the Web servers (short of manual
``env.host_string`` manipulation) was to call both as top level tasks::

    $ fab migrate update

Fabric >=1.3 can use `~fabric.tasks.execute` to set up a meta-task. Update the
``import`` line like so::

    from fabric.api import run, roles, execute

and append this to the bottom of the file::

    def deploy():
        execute(migrate)
        execute(update)

That's all there is to it; the `~fabric.decorators.roles` decorators will be honored as expected, resulting in the following execution sequence:

* `migrate` on `db1`
* `migrate` on `db2`
* `update` on `web1`
* `update` on `web2`
* `update` on `web3`

.. warning::
    This technique works because tasks that themselves have no host list (this
    includes the global host list settings) only run one time. If used inside a
    "regular" task that is going to run on multiple hosts, calls to
    `~fabric.tasks.execute` will also run multiple times, resulting in
    multiplicative numbers of subtask calls -- be careful!

    If you would like your `execute` calls to only be called once, you
    may use the `~fabric.decorators.runs_once` decorator.

.. seealso:: `~fabric.tasks.execute`, `~fabric.decorators.runs_once`


.. _leveraging-execute-return-value:

Leveraging ``execute`` to access multi-host results
---------------------------------------------------

In nontrivial Fabric runs, especially parallel ones, you may want to gather up
a bunch of per-host result values at the end - e.g. to present a summary table,
perform calculations, etc.

It's not possible to do this in Fabric's default "naive" mode (one where you
rely on Fabric looping over host lists on your behalf), but with `.execute`
it's pretty easy. Simply switch from calling the actual work-bearing task, to
calling a "meta" task which takes control of execution with `.execute`::

    from fabric.api import task, execute, run, runs_once

    @task
    def workhorse():
        return run("get my infos")

    @task
    @runs_once
    def go():
        results = execute(workhorse)
        print results

In the above, ``workhorse`` can do any Fabric stuff at all -- it's literally
your old "naive" task -- except that it needs to return something useful.

``go`` is your new entry point (to be invoked as ``fab go``, or whatnot) and
its job is to take the ``results`` dictionary from the `.execute` call and do
whatever you need with it. Check the API docs for details on the structure of
that return value.


.. _dynamic-hosts:

Using ``execute`` with dynamically-set host lists
-------------------------------------------------

A common intermediate-to-advanced use case for Fabric is to parameterize lookup
of one's target host list at runtime (when use of :ref:`execution-roles` does not
suffice). ``execute`` can make this extremely simple, like so::

    from fabric.api import run, execute, task

    # For example, code talking to an HTTP API, or a database, or ...
    from mylib import external_datastore

    # This is the actual algorithm involved. It does not care about host
    # lists at all.
    def do_work():
        run("something interesting on a host")

    # This is the user-facing task invoked on the command line.
    @task
    def deploy(lookup_param):
        # This is the magic you don't get with @hosts or @roles.
        # Even lazy-loading roles require you to declare available roles
        # beforehand. Here, the sky is the limit.
        host_list = external_datastore.query(lookup_param)
        # Put this dynamically generated host list together with the work to be
        # done.
        execute(do_work, hosts=host_list)

For example, if ``external_datastore`` was a simplistic "look up hosts by tag
in a database" service, and you wanted to run a task on all hosts tagged as
being related to your application stack, you might call the above like this::

    $ fab deploy:app

But wait! A data migration has gone awry on the DB servers. Let's fix up our
migration code in our source repo, and deploy just the DB boxes again::

    $ fab deploy:db

This use case looks similar to Fabric's roles, but has much more potential, and
is by no means limited to a single argument. Define the task however you wish,
query your external data store in whatever way you need -- it's just Python.

The alternate approach
~~~~~~~~~~~~~~~~~~~~~~

Similar to the above, but using ``fab``'s ability to call multiple tasks in
succession instead of an explicit ``execute`` call, is to mutate
:ref:`env.hosts <hosts>` in a host-list lookup task and then call ``do_work``
in the same session::

    from fabric.api import run, task

    from mylib import external_datastore

    # Marked as a publicly visible task, but otherwise unchanged: still just
    # "do the work, let somebody else worry about what hosts to run on".
    @task
    def do_work():
        run("something interesting on a host")

    @task
    def set_hosts(lookup_param):
        # Update env.hosts instead of calling execute()
        env.hosts = external_datastore.query(lookup_param)

Then invoke like so::

    $ fab set_hosts:app do_work

One benefit of this approach over the previous one is that you can replace
``do_work`` with any other "workhorse" task::

    $ fab set_hosts:db snapshot
    $ fab set_hosts:cassandra,cluster2 repair_ring
    $ fab set_hosts:redis,environ=prod status


.. _failures:

Failure handling
================

Once the task list has been constructed, Fabric will start executing them as
outlined in :ref:`execution-strategy`, until all tasks have been run on the
entirety of their host lists. However, Fabric defaults to a "fail-fast"
behavior pattern: if anything goes wrong, such as a remote program returning a
nonzero return value or your fabfile's Python code encountering an exception,
execution will halt immediately.

This is typically the desired behavior, but there are many exceptions to the
rule, so Fabric provides ``env.warn_only``, a Boolean setting. It defaults to
``False``, meaning an error condition will result in the program aborting
immediately. However, if ``env.warn_only`` is set to ``True`` at the time of
failure -- with, say, the `~fabric.context_managers.settings` context
manager -- Fabric will emit a warning message but continue executing.


.. _connections:

Connections
===========

``fab`` itself doesn't actually make any connections to remote hosts. Instead,
it simply ensures that for each distinct run of a task on one of its hosts, the
env var ``env.host_string`` is set to the right value. Users wanting to
leverage Fabric as a library may do so manually to achieve similar effects
(though as of Fabric 1.3, using `~fabric.tasks.execute` is preferred and more
powerful.)

``env.host_string`` is (as the name implies) the "current" host string, and is
what Fabric uses to determine what connections to make (or re-use) when
network-aware functions are run. Operations like `~fabric.operations.run` or
`~fabric.operations.put` use ``env.host_string`` as a lookup key in a shared
dictionary which maps host strings to SSH connection objects.

.. note::

    The connections dictionary (currently located at
    ``fabric.state.connections``) acts as a cache, opting to return previously
    created connections if possible in order to save some overhead, and
    creating new ones otherwise.

Lazy connections
----------------

Because connections are driven by the individual operations, Fabric will not
actually make connections until they're necessary. Take for example this task
which does some local housekeeping prior to interacting with the remote
server::

    from fabric.api import *

    @hosts('host1')
    def clean_and_upload():
        local('find assets/ -name "*.DS_Store" -exec rm '{}' \;')
        local('tar czf /tmp/assets.tgz assets/')
        put('/tmp/assets.tgz', '/tmp/assets.tgz')
        with cd('/var/www/myapp/'):
            run('tar xzf /tmp/assets.tgz')

What happens, connection-wise, is as follows:

#. The two `~fabric.operations.local` calls will run without making any network
   connections whatsoever;
#. `~fabric.operations.put` asks the connection cache for a connection to
   ``host1``;
#. The connection cache fails to find an existing connection for that host
   string, and so creates a new SSH connection, returning it to
   `~fabric.operations.put`;
#. `~fabric.operations.put` uploads the file through that connection;
#. Finally, the `~fabric.operations.run` call asks the cache for a connection
   to that same host string, and is given the existing, cached connection for
   its own use.

Extrapolating from this, you can also see that tasks which don't use any
network-borne operations will never actually initiate any connections (though
they will still be run once for each host in their host list, if any.)

Closing connections
-------------------

Fabric's connection cache never closes connections itself -- it leaves this up
to whatever is using it. The :doc:`fab <fab>` tool does this bookkeeping for
you: it iterates over all open connections and closes them just before it exits
(regardless of whether the tasks failed or not.)

Library users will need to ensure they explicitly close all open connections
before their program exits. This can be accomplished by calling
`~fabric.network.disconnect_all` at the end of your script.

.. note::
    `~fabric.network.disconnect_all` may be moved to a more public location in
    the future; we're still working on making the library aspects of Fabric
    more solidified and organized.

Multiple connection attempts and skipping bad hosts
---------------------------------------------------

As of Fabric 1.4, multiple attempts may be made to connect to remote servers
before aborting with an error: Fabric will try connecting
:ref:`env.connection_attempts <connection-attempts>` times before giving up,
with a timeout of :ref:`env.timeout <timeout>` seconds each time. (These
currently default to 1 try and 10 seconds, to match previous behavior, but they
may be safely changed to whatever you need.)

Furthermore, even total failure to connect to a server is no longer an absolute
hard stop: set :ref:`env.skip_bad_hosts <skip-bad-hosts>` to ``True`` and in
most situations (typically initial connections) Fabric will simply warn and
continue, instead of aborting.

.. versionadded:: 1.4

.. _password-management:

Password management
===================

Fabric maintains an in-memory, two-tier password cache to help remember your
login and sudo passwords in certain situations; this helps avoid tedious
re-entry when multiple systems share the same password [#]_, or if a remote
system's ``sudo`` configuration doesn't do its own caching.

The first layer is a simple default or fallback password cache,
:ref:`env.password <password>` (which may also be set at the command line via
:option:`--password <-p>` or :option:`--initial-password-prompt <-I>`). This
env var stores a single password which (if non-empty) will be tried in the
event that the host-specific cache (see below) has no entry for the current
:ref:`host string <host_string>`.

:ref:`env.passwords <passwords>` (plural!) serves as a per-user/per-host cache,
storing the most recently entered password for every unique user/host/port
combination (**note** that you must include **all three values** if modifying
the structure by hand - see the above link for details). Due to this cache,
connections to multiple different users and/or hosts in the same session will
only require a single password entry for each. (Previous versions of Fabric
used only the single, default password cache and thus required password
re-entry every time the previously entered password became invalid.)

Depending on your configuration and the number of hosts your session will
connect to, you may find setting either or both of these env vars to be useful.
However, Fabric will automatically fill them in as necessary without any
additional configuration.

Specifically, each time a password prompt is presented to the user, the value
entered is used to update both the single default password cache, and the cache
value for the current value of ``env.host_string``.

.. [#] We highly recommend the use of SSH `key-based access
    <http://en.wikipedia.org/wiki/Public_key>`_ instead of relying on
    homogeneous password setups, as it's significantly more secure.


.. _ssh-config:

Leveraging native SSH config files
==================================

Command-line SSH clients (such as the one provided by `OpenSSH
<http://openssh.org>`_) make use of a specific configuration format typically
known as ``ssh_config``, and will read from a file in the platform-specific
location ``$HOME/.ssh/config`` (or an arbitrary path given to
:option:`--ssh-config-path`/:ref:`env.ssh_config_path <ssh-config-path>`.) This
file allows specification of various SSH options such as default or per-host
usernames, hostname aliases, and toggling other settings (such as whether to
use :ref:`agent forwarding <forward-agent>`.)

Fabric's SSH implementation allows loading a subset of these options from one's
actual SSH config file, should it exist. This behavior is not enabled by
default (in order to be backwards compatible) but may be turned on by setting
:ref:`env.use_ssh_config <use-ssh-config>` to ``True`` at the top of your
fabfile.

If enabled, the following SSH config directives will be loaded and honored by Fabric:

* ``User`` and ``Port`` will be used to fill in the appropriate connection
  parameters when not otherwise specified, in the following fashion:

  * Globally specified ``User``/``Port`` will be used in place of the current
    defaults (local username and 22, respectively) if the appropriate env vars
    are not set.
  * However, if :ref:`env.user <user>`/:ref:`env.port <port>` *are* set, they
    override global ``User``/``Port`` values.
  * User/port values in the host string itself (e.g. ``hostname:222``) will
    override everything, including any ``ssh_config`` values.
* ``HostName`` can be used to replace the given hostname, just like with
  regular ``ssh``. So a ``Host foo`` entry specifying ``HostName example.com``
  will allow you to give Fabric the hostname ``'foo'`` and have that expanded
  into ``'example.com'`` at connection time.
* ``IdentityFile`` will extend (not replace) :ref:`env.key_filename
  <key-filename>`.
* ``ForwardAgent`` will augment :ref:`env.forward_agent <forward-agent>` in an
  "OR" manner: if either is set to a positive value, agent forwarding will be
  enabled.
* ``ProxyCommand`` will trigger use of a proxy command for host connections,
  just as with regular ``ssh``.

  .. note::
    If all you want to do is bounce SSH traffic off a gateway, you may find
    :ref:`env.gateway <gateway>` to be a more efficient connection method
    (which will also honor more Fabric-level settings) than the typical ``ssh
    gatewayhost nc %h %p`` method of using ``ProxyCommand`` as a gateway.

  .. note::
    If your SSH config file contains ``ProxyCommand`` directives *and* you have
    set :ref:`env.gateway <gateway>` to a non-``None`` value, ``env.gateway``
    will take precedence and the ``ProxyCommand`` will be ignored.

    If one has a pre-created SSH config file, rationale states it will be
    easier for you to modify ``env.gateway`` (e.g. via
    `~fabric.context_managers.settings`) than to work around your conf file's
    contents entirely.
