====
使用 库
====

Fabric的主要用法是通过fabfile和 :doc:`fab </usage/fab>` 工具, 在很多文档里皆有体现.
然而, Fabric的内部被一种方式来容易的使用而不是通过 ``fab`` 和fabfiles 这篇文档将告诉你
怎样做.

这里确实有两个因素之一需要牢记, 相比编写fabfile或使用 ``fab`` 去运行，如何进行连接和如何断开
连接.

连  接
====

之前我们已经说明了Fabric如何连接到主机, 但是目前它有些被xxx
:doc:`execution docs </usage/execution>`. 特别的, 你可能想要跳过 :ref:`connections`
部分能够更快的阅读. (你真的应该给文档一个结尾，但不是绝对必要的.)

作为这一部分的提醒, 关键点仅仅是 `~fabric.operations.run`, `~fabric.operations.sudo`
其他的操作在and the other operations only look in one place when
connecting: :ref:`env.host_string <host_string>`. 所有其他为了设定主机的机制
All of the other mechanisms
for setting hosts are interpreted by the ``fab`` tool when it runs, and don't
matter when running as a library.

That said, most use cases where you want to marry a given task ``X`` and a given list of hosts ``Y`` can, as of Fabric 1.3, be handled with the `~fabric.tasks.execute` function via ``execute(X, hosts=Y)``. Please see `~fabric.tasks.execute`'s documentation for details -- manual host string manipulation should be rarely necessary.

关闭连接
====

The other main thing that ``fab`` does for you is to disconnect from all hosts
at the end of a session; otherwise, Python will sit around forever waiting for
those network resources to be released.

Fabric 0.9.4 and newer have a function you can use to do this easily:
`~fabric.network.disconnect_all`. Simply make sure your code calls this when it
terminates (typically in the ``finally`` clause of an outer ``try: finally``
statement -- lest errors in your code prevent disconnections from happening!)
and things ought to work pretty well.

If you're on Fabric 0.9.3 or older, you can simply do this (``disconnect_all``
just adds a bit of nice output to this logic)::

    from fabric.state import connections

    for key in connections.keys():
        connections[key].close()
        del connections[key]


最后的提醒
=====

本文档This document is an early draft, and may not cover absolutely every difference
between ``fab`` use and library use. However, the above should highlight the
largest stumbling blocks. When in doubt, note that in the Fabric source code,
``fabric/main.py`` contains the bulk of the extra work done by ``fab``, and may
serve as a useful reference.
