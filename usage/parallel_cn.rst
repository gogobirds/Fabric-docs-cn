====
并发执行
====

.. _parallel-execution:

.. versionadded:: 1.3

Fabric会默认**连续**执行所有的指定任务(更多细节参见 :ref:`execution-strategy`). 这篇文档描述了
Fabric通过每个任务的修饰符 和/或 全局命令行开关, 在多个主机上**并发**执行任务的选项.


它做什么
====

因为Fabric 1.x没有完全保证线程安全(还因为一般使用时, 任务函数不会互相作用), 这个功能通过Python
 `multiprocessing <http://docs.python.org/library/multiprocessing.html>`_ 模块实现.
它为每个主机和任务组合创建一个新进程, 有选择地使用(可配置)滑动窗口以防止太多进程同时运行.

例如, 想象这样一个场景: 你想要在很多Web服务器上更新Web应用代码, 一旦代码全部更新好就重新加载服务
(万一代码更新失败就可以很容易地还原.) 可以通过以下fabfile实现::

    from fabric.api import *

    def update():
        with cd("/srv/django/myapp"):
            run("git pull")

    def reload():
        sudo("service apache2 reload")

在一组的3个服务器上串行执行, 比如这样::

    $ fab -H web1,web2,web3 update reload

通常情况下, 没有激活任何并发执行选项的话, Fabric会按顺序执行:

#. ``update`` on ``web1``
#. ``update`` on ``web2``
#. ``update`` on ``web3``
#. ``reload`` on ``web1``
#. ``reload`` on ``web2``
#. ``reload`` on ``web3``

激活并发执行选项的话 (通过 :option:`-P` -- 更多细节参见下文), 又会像下面这样:

#. ``update`` on ``web1``, ``web2``, and ``web3``
#. ``reload`` on ``web1``, ``web2``, and ``web3``

希望这所带来的好处是显而易见的 -- 如果 ``update`` 用了5秒来运行并且 ``reload`` 用了2秒,
串行执行会花费 (5+2)*3 = 21 秒, 而并发执行平均只用了三分之一的时间, (5+2) = 7 秒.


如何使用
====

装饰器
---

既然并发执行作用的最小"单元"是一个任务, 这个功能可以通过 `~fabric.decorators.parallel`
和 `~fabric.decorators.serial` 装饰器逐个任务地启用或禁用. 例如, 下面这个fabfile::

    from fabric.api import *

    @parallel
    def runs_in_parallel():
        pass

    def runs_serially():
        pass

当以这种方式运行::

    $ fab -H host1,host2,host3 runs_in_parallel runs_serially

会带来以下的执行顺序:

#. ``runs_in_parallel`` on ``host1``, ``host2``, and ``host3``
#. ``runs_serially`` on ``host1``
#. ``runs_serially`` on ``host2``
#. ``runs_serially`` on ``host3``

命令行标志
-----

或许有人会希望使用命令行标志 :option:`-P` 或者环境变量 :ref:`env.parallel <env-parallel>`来并发执行所有任务.
然而, 任何使用`~fabric.decorators.serial` 特定封装的任务都会忽略掉这个设置并且继续串行执行.

例如, 以下的fabfile会带来和以上例子一样的执行顺序::

    from fabric.api import *

    def runs_in_parallel():
        pass

    @serial
    def runs_serially():
        pass

当如下调用时::

    $ fab -H host1,host2,host3 -P runs_in_parallel runs_serially

``runs_in_parallel``仍旧会并发执行,  ``runs_serially`` 也会按照顺序.


并发池大小
=====

有了太大的主机列表, 用户的本地机器可能会因运行大量的Fabric进程而超负荷工作.
因此, 你可能会选择使用moving bubble的方法以限制Fabric同时运行特定数量的活跃进程.

默认不使用任何的bubble, 所有主机也都运行在一个并发池. 你可以在任务级别通过
指定 ``pool_size`` 的关键参数或全局地通过 :option:`-z`来覆盖.

比如, 同时在5个主机上运行::

    from fabric.api import *

    @parallel(pool_size=5)
    def heavy_task():
        # lots of heavy local lifting or lots of IO here

或者跳过 ``pool_size`` kwarg并且如下::

    $ fab -P -z 5 heavy_task

.. _linewise-output:

行vs字节 输出
========

Fabric的输出到终端的默认模式是逐字节的, 以便支持 :doc:`/usage/interactivity`.
这样当以并发模式执行时往往效果不好, 因为多个进程可能会同时写进你的终端标准输出流.

为了抵消这个问题, Fabric的linewise输出选项会在并发模式激活时自动启用. 这会导致你失去以上链接中
Fabric远程交互特性里的大部分好处, 但是那些不会很好地映射到并发调用, 这是很公平的损益交易.

逐行地基础上, 没有办法避免多个进程的混合, 但至少你可以通过主机字符串一行的前缀来区分它们.

.. note::
    以后的的版本将会改进日志功能以便更容易地检修并发执行.
