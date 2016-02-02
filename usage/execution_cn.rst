====
执行模型
====

如果你已经读过 :doc:`../tutorial`, 你应该熟悉对Fabric的基本使用(单主机上执行单任务) 然而,
在很多情况下,你会发现需要在单/多个主机上执行多个任务. 也许你想把一个大任务分割为小的部分,
或者删除一组服务器上的老用户. 这种情景下,何时及如何执行任务都有特定的规则.

本文档将探讨Fabric的执行模型,包括主执行循环,如何定义主机列表,如何连接等.


.. _execution-strategy:

执行策略
====

Fabric默认采用串行执行单任务的方式, 虽然在Fabric 1.3中可以采用并行模式来替代
(参见 :doc:`/usage/parallel`). 默认行为如下：

* 创建一个任务列表,简单地按照列表顺序作为 :doc:`fab <fab>` 的参数即可.
* 对于每一个任务,从各种来源生成一个特定任务主机列表.(参见 :ref:`host-lists` ).
* 任务列表按照顺序执行, 在主机列表内的每个主机上, 每个任务只执行一次.
* 主机列表没有列出执行主机的任务默认为在本地执行,且只运行一次.

如此一来, 给出以下fabfile::

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

虽然这种方式很简单, 能够将任务简单地组合到一起（不像发送任务到多个主机分别调用的工具）,同时可以使用脚本一样的逻辑,
以便从输出得到反馈或得到命令的返回码,从而决定下一步做什么


定义任务
====

关于Fabric任务的构成和组建,参见:doc:`/usage/tasks`.

定义主机列表
======

除非你只是在简单执行任务构建的系统上使用Fabric（可以但不是基本用法）,
"构建任务却不指定特定的远程主机去执行任务"这种做法没有任何好处.
有很多方法去指定远程主机,从全局到每个任务都可以根据需要组合和匹配.

.. _host-strings:

主  机
----

主机,在这种上下文中通常也被称为"主机字符串": 一个由用户名,主机名和端口组合而成的Python
字符串,如``username@hostname:port``这种形式,用户和端口可以被省略（由``@`` 或 ``:``
所关联）,由本地用户名和默认端口22来代替.因此``admin@foo.com:222``, ``deploy@website``
和 ``nameserver1`` 等都是有效的主机串.

同时也支持IPv6, 比如 ``::1``, ``[::1]:1222``, ``user@2001:db8::1`` 或 ``user@[2001:db8::1]:1222``.
有必要使用方括号作为地址与端口的分隔符.如果不需要端口号,方括号可选.
如果通过命令行参数指定主机字符串,在一些shell中可能需要转义方括号.

.. 注意::
    用户和主机通过最后一个``@``符号分隔,所以email作为用户名是有效的,会被正确解析.

执行时,Fabric格式化主机串并储存其每个部分（用户名/主机名/端口号）到环境字典中,以便使用或任务中需要的调用.
详情参见:doc:`env`.

.. _execution-roles:

角  色
----

主机串对应单个主机,有时将主机分组会很有用.也许你想要更新一个负载均衡器后的多个Web服务器,
或者你想要在所有客户端服务器上执行同一任务.
角色提供一种定义字符串以对应到主机串列表的的功能,以便快速指定而非每次写出完整的主机列表.

该映射定义为字典 ``env.roledefs``, 必须通过修改fabfile来使用.
简单的例子::

    from fabric.api import env

    env.roledefs['webservers'] = ['www1', 'www2', 'www3']

由于 ``env.roledefs`` 默认为空,你可以重新定义它而不必担心丢失任何信息
(前提是你不加载其他fabfiles去修改它)::

    from fabric.api import env

    env.roledefs = {
        'web': ['www1', 'www2', 'www3'],
        'dns': ['ns1', 'ns2']
    }

角色的定义不一定是主机的配置,也可根据所需执行其他环境变量的特定配置.
通过定义角色字典和在``hosts``键下定义主机串实现::

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

除了列表/可迭代类型,``env.roledefs``的值（或字典样式里定义``hosts``键的值）也是可调用的,
当任务运行而非模块加载时被调用.(例如,你可以在连接远程服务器时获取角色定义,
且不必担心在调用``fab --list``时加载fabfile引起延迟)

任何情况下,都不用必须使用角色 -- 在你有通用的服务器分组时,它只不过是提供了一种便利的方式

.. 版本变更:: 0.9.2
    增加可调用的值``roledefs``.

.. _host-lists:

如何构造主机列表
--------

有很多种方式来指定主机列表,无论是全局使用还是单个任务中调用,通常这些方法会被互相覆盖而不是合并到一起
(虽然有可能在未来的版本变更).每种方式都被分为两部分,用于主机和用于角色.

全局变量, 通过 ``env``
~~~

修改环境字典的两个键值对是设定主机或角色最通用的方式,:doc:`env <env>`: ``hosts`` 和 ``roles``.
运行程序并构建每个任务的主机列表时,会检查这些变量的值.

因此,通过fabfile被导入时会被设定为模块级别的变量::

    from fabric.api import env, run

    env.hosts = ['host1', 'host2']

    def mytask():
        run('ls /var/www')

像这样的fabfile,通过``fab mytask``运行,便会依次在``host1``,``host2``上执行``mytask``.

由于*每个* 任务都会检查环境变量,这就意味着你可以根据需要在一个任务中修改``env``, 而这样做会影响到所有后续任务::

    from fabric.api import env, run

    def set_hosts():
        env.hosts = ['host1', 'host2']

    def mytask():
        run('ls /var/www')

通过``fab set_hosts mytask``运行时, ``set_hosts`` 是一个"本地"任务 -- 它原本的主机列表为空
-- 但 ``mytask`` 会在给出的两个主机中运行.

.. note::
    这种方式常用于创建虚拟的角色,但角色被完全定义时便很少需要这样.这种方式在以后的一些情形中可能还会有用.

``env.hosts`` 作为 ``env.roles`` (不要和``env.roledefs``混淆!)
被给出时,会作为角色名列表在 ``env.roledefs``中查找.

全局变量, 通过命令行参数
~~~

除了在模块级别修改 ``env.hosts``, ``env.roles`` 和 ``env.exclude_hosts`` ,也可以通过
命令行选项 :option:`--hosts/-H <-H>` 和 :option:`--roles/-R <-R>`的逗号分隔式字符串参数来定义它们
例如.::

    $ fab -H host1,host2 mytask

这种调用相当于 ``env.hosts = ['host1', 'host2']`` -- 参数解析器在解析时会寻找这些参数并修改``env``

.. note::

    事实上,可能会使用这些选项设定单个主机或角色,Fabric在得到字符串时调用的是``string.split(',')``,
    所以没有逗号的字符串会变成单个列表.

一定要知道————命令行选项会在加载你的fabfile前被解释:在fabfile里对``env.hosts`` 或
``env.roles``的任何重定义都会覆写它们.


如果希望命令行定义和fabfile定义的主机列表无损合并,确保在fabfile中使用``env.hosts.extend()``::

    from fabric.api import env, run

    env.hosts.extend(['host3', 'host4'])

    def mytask():
        run('ls /var/www')

通过 ``fab -H host1,host2 mytask``运行这个fabfile, 那么在``mytask``执行时,
``env.hosts`` 将会包含``['host1', 'host2', 'host3', 'host4']``.

.. note::

    ``env.hosts`` 就是一个Python列表对象 -- 所以可以使用``env.hosts.append()``
    或者其他你想用的列表方法.

.. _hosts-per-task-cli:

单任务,通过命令行参数
~~~~~~~~~~~

设置全局主机列表足以一直让所有任务跑在相同的主机上.但有时不需要这样,所以Fabric提供一些方法
更精确、特例地指定单个任务的主机列表.第一个方法是指定任务参数.

如:doc:`fab` 所述, 可以通过特定的命令行语法指定任务参数. 除了命名任务的实际参数,还可以设定
``host``, ``hosts``, ``role`` 或 ``roles`` 等"参数",并在Fabric建立主机列表时被解析
(自动从传递给任务的参数中删除.)

.. note::

    由于逗号一直被用来分割任务参数,而分号用来划分``hosts`` 或 ``roles`` 以便描述主机串和角色名.
    此外,必须加上引号以防shell解析分号.

运行下面的fabfile, 除了没有定义主机信息，其余和使用过的都一样::

    from fabric.api import run

    def mytask():
        run('ls /var/www')

为``mytask``指定特定的主机, 按以下执行::

    $ fab mytask:hosts="host1;host2"

它将覆盖所有的主机列表并确保``mytask``只在这两个主机上执行.

单任务, 通过装饰器
~~~~~~~~~~

如果一个任务总是根据预定义的主机列表运行,你可能希望在fabfile中指定.
可通过`~fabric.decorators.hosts` 或 `~fabric.decorators.roles`装饰任务函数.
装饰器需要一个可变的参数列表,例如::

    from fabric.api import hosts, run

    @hosts('host1', 'host2')
    def mytask():
        run('ls /var/www')

也可以传入一个可迭代的参数 如::

    my_hosts = ('host1', 'host2')
    @hosts(my_hosts)
    def mytask():
        # ...

在使用时,这些装饰器覆盖了``env``对特定主机列表的检查 (实际上没有对``env``进行任何修改,只是简单地忽略.)
因此,即使fabfile定义了 ``env.hosts``或者通过选项 :option:`--hosts/-H <-H>` 调用 :doc:`fab <fab>`
``mytask``仍然只会在``['host1', 'host2']``上执行.

然而,装饰主机列表**不会**覆盖单个任务的命令行参数,前一节已解释过.

优 先级
~~~~

我们已经一起研究了设定主机列表的方法,然而,为了更清晰,快速回顾一下:

* 单任务,通过命令行(``fab mytask:host=host1``),可覆盖其他所有方法.
* 单任务,通过装饰特定主机列表(``@hosts('host1')``),可覆盖``env`` 变量.
* 在fabfile中全局指定主机列表(``env.hosts = ['host1']``)*可以* 覆盖通过命令行指定的列表
  但只会在你疏漏（或希望）的情况下.
* 在命令行中全局指定主机列表(``--hosts=host1``),将会初始化``env``变量,仅此而已.

这个逻辑顺序可能会在未来版本中轻微变动以达更加一致 (例如,使用选项 :option:`--hosts <-H>`在某种程度上
优先于``env.hosts``使用命令行指定同一个单任务）但只会在向后兼容的版本中出现.

.. _combining-host-lists:

结合主机列表
------

根据 :ref:`host-lists` 在不同的方法中没有提及主机的"合并".
如果``env.hosts``设置为``['host1', 'host2', 'host3']``, 单个函数(例如. 通过 `~fabric.decorators.hosts`)
把主机列表设置为 ``['host2', 'host3']``,则函数将**不会**在``host1``执行,因为单任务的装饰器优先级较高.

然而,对于每个给定源,如果角色和主机**同时**被指定,他们将会被合并到一个主机列表,
例如下面这个fabfile,同时使用两个装饰器::

    from fabric.api import env, hosts, roles, run

    env.roledefs = {'role1': ['b', 'c']}

    @hosts('a', 'b')
    @roles('role1')
    def mytask():
        run('ls /var/www')

如果执行 ``mytask`` 时没有给予命令行参数,这个fabfile将会调用``mytask``在``['a', 'b', 'c']``上执行
 -- 结合 ``role1`` 和 `~fabric.decorators.hosts` 的调用.

.. _deduplication:

主机列表去重
------

默认支持去重 :ref:`combining-host-lists`, Fabric对最终的主机列表去重保证主机只出现一次.
无论如何,这将防止明确/故意地在同一个目标主机多次执行一个任务,有时候是有用的.

关掉主机去重,可设置 :ref:`env.dedupe_hosts <dedupe_hosts>` 为 ``False``.


.. _excluding-hosts:

不包括特定主机
-------

有时候,不包括一个或多个特定主机是有用的,例如,
覆盖一些从角色或自动生成的主机列表中得来的无法正常工作或意料之外的主机.

.. note::
    在Fabric 1.4, 可能希望使用 :ref:`skip-bad-hosts` 来自动跳过不可访问的主机.

通过全局选项去掉主机 :option:`--exclude-hosts/-x <-x>`::

    $ fab -R myrole -x host2,host5 mytask

如果 ``myrole`` 被定义为 ``['host1', 'host2', ..., 'host15']``,下面的调用将会运行
在 ``['host1', 'host3','host4', 'host6', ..., 'host15']`` 中有效的主机列表上.

    .. note::
        使用这个选项不会修改 ``env.hosts`` -- 仅仅会引起主执行循环去掉请求的主机.

通过使用额外的``exclude_hosts``变量从特定的单个任务中去除,
它的执行类似于上述 ``hosts`` 和 ``roles``的单任务情况,也是从实际任务中剥离.这
个例子和全局去除有相同的结果::

    $ fab mytask:roles=myrole,exclude_hosts="host2;host5"

注意主机列表由逗号分割,如 ``hosts`` 中的单任务参数.

结合去除
~~~~

主机去除列表类似于主机列表本身,不会通过可以进行声明的"层级"互相覆盖.
Host exclusion lists, like host lists themselves, are not merged together
across the different "levels" they can be declared in. 例如,一个全局选项
``-x`` 将不会影响一个通过装饰器或命令参数设定的单任务主机列表,单任务的``exclude_hosts``参数
也不会影响全局 ``-H`` 列表.

这个规则有一个小的例外,即CLI级别关键字参数 (``mytask:exclude_hosts=x,y``)
通过 ``@hosts`` 或 ``@roles``检查主机列表设置时**将**会被考虑在内.
因此被 ``@hosts('host1', 'host2')`` 装饰的任务函数以命令 ``fab taskname:exclude_hosts=host2``
执行时只会在 ``host1``上运行.

由于主机列表合并,这个功能在当前版本被限制 (一定程度上保持了执行的简单) 并可能在将来的版本进行扩展.


.. _execute:

使用 ``execute`` 智能执行任务
=====================

.. versionadded:: 1.3

根据 :doc:`fab <fab>` 涉及 "top level" 任务的执行,就像第一个例子中我们调用
``fab taskA taskB``. 当然,很容易的包装成多任务调用,"元"任务.

在Fabric 1.3之前, 这是很难完成的, 在概述 :doc:`/usage/library`中. Fabric的设计避开了神奇的行为,
所以正如 `~fabric.decorators.roles` 中所述, **调用** 一个任务函数 **不会** 理会装饰器.

在Fabric 1.3增加的是 `~fabric.tasks.execute` 辅助函数, 需要一个任务对象或任务名称作为第一个参数.
等效于从命令行调用给定的任务:所有给出的规则在 :ref:`host-lists`适用.
( ``hosts`` 和 ``roles`` 对于 `~fabric.tasks.execute`的关键字参数与
 :ref:`CLI per-task arguments<hosts-per-task-cli>`类似,包括他们如何与主机/角色设置的方法彼此覆盖)

举例说明,如下的fabfile定义了两个部署Web应用的独立的任务::

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

在1.2以下版本的Fabric中, 确保 ``migrate`` 运行在DB服务器和 ``update`` 运行在Web服务器 (短手册 ``env.host_string``)
的唯一方法是同时调用两个顶级任务::

    $ fab migrate update

在1.3及以上版本的Fabric中,可以使用 `~fabric.tasks.execute` 设置元任务. 更新 ``import`` 一行如下::

    from fabric.api import run, roles, execute

然后在文件底部增加下面的语句::

    def deploy():
        execute(migrate)
        execute(update)

事情都搞定了, `~fabric.decorators.roles` 装饰器将被如期执行, 所得执行顺序如下:

* `migrate` on `db1`
* `migrate` on `db2`
* `update` on `web1`
* `update` on `web2`
* `update` on `web3`

.. warning::
    这种技术的工作原理是因为任务本身没有只运行一次主机列表 (包括全局主机列表的设定).
    如果使用"定时"任务将会运行在多个主机,调用 `~fabric.tasks.execute` 也会运行多次,
    导致多个子任务以倍数级的次数调用 -- 需要小心!

    如果你想要 `execute` 仅仅执行一次, 可以使用 `~fabric.decorators.runs_once` 装饰器.

.. seealso:: `~fabric.tasks.execute`, `~fabric.decorators.runs_once`


.. _leveraging-execute-return-value:

利用 ``execute`` 访问多主机结果
---

当Fabric不一般运行,特别是并发执行时,你可能需要得到每个主机的最终结果值 - 比如形成汇总表,
执行计算等.

Fabric的默认模式（即通过Fabric循环遍历你的主机列表）不能做到,但是使用
`.execute` 能轻松解决.只需要从实际调用的任务中切换到调用"元"任务,以控制 `.execute`
执行::

    from fabric.api import task, execute, run, runs_once

    @task
    def workhorse():
        return run("get my infos")

    @task
    @runs_once
    def go():
        results = execute(workhorse)
        print results

如上, ``workhorse`` 可以做Fabirc的所有事 -- 它和"原生"任务表面上是一样的
-- 除了它会返回一些有用的东西.

``go`` 是一个新的入口点 (通过 ``fab go`` 来调用,或者诸如此类的东西) 通过 `.execute`
调用来接受返回值 ``results`` 字典,以便完成任何你所需要的. 查阅 API 文档查看关于返回值结构
的更多细节.

.. _dynamic-hosts:

使用 ``execute`` 动态设定主机列表
-----------------------

Fabirc一个常见的从中级到高级的用法是在运行时使用参数查询目标主机列表
(使用 :ref:`execution-roles` 不足以完成). ``execute`` 能够非常简单的做到,如下::

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

如例子, 如果 ``external_datastore`` 是一个简单的 "通过标签从数据库查询主机" 的功能.
你想要运行一个任务在所有的主机被应用堆栈联系起来,你可以这样调用它::

    $ fab deploy:app

等一等! 一旦数据库服务器的数据发生迁移,可以通过源来修复我们的迁移代码,然后仅仅再次通过db部署::

    $ fab deploy:db

这种使用方法看起来类似于Fabric的roles,但是有更多可挖掘的,不意味这限制了单个参数.
无论你希望如何定义任务,查询你的外部数据通过你想要的办法 -- 它就是Python.

另一种办法
~~~

类似上述,但是使用 ``fab`` 能够调用多个任务而不是明确的 ``execute`` 调用,
:ref:`env.hosts <hosts>` 在主机列表查找任务然后调用 ``do_work`` 在同一个会话中::

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

接着如下调用::

    $ fab set_hosts:app do_work

比起前一种的方法的好处是你可以用 ``do_work`` 来代替任何一个 "workhorse" 任务::

    $ fab set_hosts:db snapshot
    $ fab set_hosts:cassandra,cluster2 repair_ring
    $ fab set_hosts:redis,environ=prod status


.. _failures:

故障处理
====

一旦任务列表构建完成,Fabirc像概述 :ref:`execution-strategy` 中那样开始执行
直到所有任务在全部的主机列表上运行完成. 然而,Fabirc默认使用一种 "快速失败" 的匹配行为
一旦发生任何错误,比如一个远程程序返回一个非零的返回值或者fabfile的Python代码发生了一个异常,
执行将会立即停止.

这通常是期望的行为,但也有很多的例外,所以Fabric提供 ``env.warn_only``,一个布尔值设定.
默认为 ``False``,意味这一个错误条件的发生将导致程序立刻终止执行.但是,如果 ``env.warn_only``
被设置为 ``True`` 也就是说在 `~fabric.context_managers.settings` 上下文管理器为 ``True``
Fabirc 在失败时会发出警告信息但继续执行.

.. _connections:

连接
==

``fab`` 命令自身并不能连接到远程主机,它只是确保每个任务在它的主机列表上执行,环境变量
``env.host_string`` 设置为正确值. 如果用户想要利用Fabric作为一个库可手动操作来实现相同
的效果 (即使在Fabirc 1.3以后,使用`~fabric.tasks.execute` 是更强大的选择.)

``env.host_string`` 是 (顾名思义) 一个当前的主机,当网络相关函数运行时Fabric用来确定使用什么样去连接.
像操作  `~fabric.operations.run` 或 `~fabric.operations.put` 使用 ``env.host_string`` 在一个
映射主机到SSH连接对象的共享的字典中作为查找关键字.

.. note::

    该连接字典(位于 ``fabric.state.connections``) 作为缓存,如果可能的话选择加入先前创建的连接以节省开销,
    否则才创建新的连接.

慢连接
---

因为连接是被各个业务所驱动的,Farbic也不会使用连接除非需要.一个例子,这个任务
在与远程服务器交互之前进行了本地操作::

    from fabric.api import *

    @hosts('host1')
    def clean_and_upload():
        local('find assets/ -name "*.DS_Store" -exec rm '{}' \;')
        local('tar czf /tmp/assets.tgz assets/')
        put('/tmp/assets.tgz', '/tmp/assets.tgz')
        with cd('/var/www/myapp/'):
            run('tar xzf /tmp/assets.tgz')

发生了什么, 从连接的的角度看如下:

#. 两个 `~fabric.operations.local` 调用运行不会做任何关于网络连接的事;
#. `~fabric.operations.put` 从连接缓存请求一个连接到 ``host1``;
#. 不能找到已存在的连接从主机,所以会创建一个新的SSH连接,返回给 `~fabric.operations.put`;
#. `~fabric.operations.put` 通过这个连接上传文件;
#. 最后, `~fabric.operations.run` 从连接缓存请求相同的主机,因为已经存在,直接使用缓存.

从此可看出,你任何不使用网络连接的操作将不会真正启动连接(虽然他们会在主机列表上的
每个主机运行一次, 如果有的话)

关闭连接
----

Fabric不会自己关闭连接缓存 -- 它将一直存在无论是否使用.
:doc:`fab <fab>` 工具为你做了记录: 它遍历所有打开的连接在退出之前关闭
(不管任务是否执行成功.)

库的使用者需要确保他们所有打开的连接明确关闭在程序退出之前. 可以通过在脚本最后调用
`~fabric.network.disconnect_all` 来完成.

.. note::
    `~fabric.network.disconnect_all` 在未来版本可能被移动到更公开的定位;我们仍然在
    努力使Fabirc的库更加固定和有组织.d.

多次连接重试和跳过失败主机
-------------

在Fabric 1.4,多次尝试连接可能在错误终止前连接到远程服务器: Fabric尝试连接
:ref:`env.connection_attempts <connection-attempts>` 次才会放弃,
每次会等待超时时间 :ref:`env.timeout <timeout>` 秒. (默认从1到10秒以配合以前的行为,
但是他们可以安全的转换为你需要的状态.)

此外, 即使永远不能连接到服务器也不会被强制终止: 设置 :ref:`env.skip_bad_hosts <skip-bad-hosts>`
为 ``True`` Fabric使大多数情况 (通常为初识连接) 只会警告并继续,而不是终止.

.. versionadded:: 1.4

.. _password-management:

密码管理
====

Fabric保持在内存中,两层密码能够帮助你记住登陆和sudo密码在某些情况下; 这有助于避免
多次尝试当多个系统使用同一个密码 [#]_, 如果一个远程系统 ``sudo`` 配置不执行自己的缓存.

第一层是简单的默认或者回调密码缓存 :ref:`env.password <password>` (可通过命令行
:option:`--password <-p>` or :option:`--initial-password-prompt <-I>` 设定).
该环境变量储存一个简单的密码 (如果不为空) 将会尝试使用主机专用缓存 (即将说明) 在没有入口时.

:ref:`env.passwords <passwords>` (复数!) 提供一个用户/主机对的缓存
储存最近输入的密码为每个不相同的用户/主机/端口组合 (**注意** 你必须包括 **三个值**
如果手动修改结构 - 详情请参见上面的链接). 由于这个缓存, 连接到不同的用户和(或)主机用相同的
会话将仅仅需要一个密码为每一个项. (先前版本的Fabirc只使用单一的,默认的密码缓存,因此
先前的密码变为无效需要重新输入密码.)

依赖与你的配置和你的会话需要连接的一些主机,你可能发现设定一些环境变量是有用的.
然而,Fabric会在必要时自动填充他们在不需要任何配置.

特别的,每次密码验证被提示给用户,输入的值用于更新单一默认的密码缓存和 ``env.host_string``
当前值的缓存.

.. [#] 我们多次提醒使用SSH密钥 `key-based access
    <http://en.wikipedia.org/wiki/Public_key>`_ 而不是依赖同质密码设置,因为那样更安全.


.. _ssh-config:

利用原生 SSH 配置文件
=============

命令行SSH客户端 (如由 `OpenSSH <http://openssh.org>`_ 提供的) 使用一个特定的配置格式
如典型的 ``ssh_config``, 根据平台特定的路径 ``$HOME/.ssh/config`` 下的一个文件读取
(或者一个任意的路径通过 :option:`--ssh-config-path`/:ref:`env.ssh_config_path <ssh-config-path>`.)
文件允许制定各个SSH选项如默认的或者某个主机的 用户名,主机名别名以及切换等其他设置
(如是否使用 :ref:`agent forwarding <forward-agent>`.)

Fabric的 SSH 实现允许从一个实际的SSH配置文件加载这些选项的子集,它们应该存在. 这个行为
不是默认开启的(为了向后兼容) 但可以通过在fabfile顶部设置 :ref:`env.use_ssh_config <use-ssh-config>`
为 ``True`` 开启.

一旦开启,下面的SSH配置指令将被Fabric加载和想回呗If enabled, the following SSH config directives will be loaded and honored by Fabric:

* ``User`` and ``Port`` 将被用于填写相应的连接参数在没有其他规定时,以下面的方式:

  * 全局指定 ``User``/``Port`` 将被当前默认代替(分别是本地用户名和22端口)
    如果适合的环境变量没有被设定.
  * 然而, 如果 :ref:`env.user <user>`/:ref:`env.port <port>` *被* 设置, 他们将
    覆盖全局的 ``User``/``Port`` 值.
  * 在主机字符串里的用户/密码 (例如. ``hostname:222``) 将覆盖全部,包括任何
    ``ssh_config`` 的值.
* ``HostName`` 可以被给予的主机名所代替,仅需要像一般的``ssh``. 因此``Host foo`` 项指定为
  ``HostName example.com`` 会允许你给予Fabric主机名 ``'foo'`` 并扩展为 ``'example.com'`` 在连接时.
* ``IdentityFile`` 将扩展 (不是代替) :ref:`env.key_filename <key-filename>`.
* ``ForwardAgent`` 将扩展 :ref:`env.forward_agent <forward-agent>` 用 "or" 的方式:
  如果任意一个被设置为正值,将会开始代理转发.
* ``ProxyCommand`` 会触发使用代理命令用于主机连接,正如常规的 ``ssh``.

  .. note::
    如果你想要做的是反弹SSH作为网关,会发现 :ref:`env.gateway <gateway>` 比一般的
    使用 ``ProxyCommand`` 作为网关``ssh gatewayhost nc %h %p`` 是一个更有效的
    连接方法 (也会实现更多Fabric级别的设置)

  .. note::
    如果你的SSH配置文件包含 ``ProxyCommand`` 指令*并且* 设置了 :ref:`env.gateway <gateway>`
    为一个 ``None`` 值, ``env.gateway`` 会被优先考虑,``ProxyCommand`` 将被忽略.

    如果一个人拥有预设的SSH配置文件,原理上能够方便的修改 ``env.gateway`` (如通过
    `~fabric.context_managers.settings`) 而不是通过是配置文件相同.
