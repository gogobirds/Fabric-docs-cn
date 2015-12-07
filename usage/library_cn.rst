====
使用 库
====

Fabric的主要用法是通过fabfile和 :doc:`fab </usage/fab>` 工具, 在很多文档里皆有体现.
然而, Fabric的内部被一种方式来容易的使用而不是通过 ``fab`` 和fabfiles 这篇文档将告诉你
怎样做.

这里确实有两个因素之一需要牢记, 相比编写fabfile或使用 ``fab`` 去运行，如何进行连接和如何断开
连接.

Connections
===========

We've documented how Fabric really connects to its hosts before, but it's
currently somewhat buried in the middle of the overall :doc:`execution docs
</usage/execution>`. Specifically, you'll want to skip over to the 
:ref:`connections` section and read it real quick. (You should really give that
entire document a once-over, but it's not absolutely required.)

As that section mentions, the key is simply that `~fabric.operations.run`,
`~fabric.operations.sudo` and the other operations only look in one place when
connecting: :ref:`env.host_string <host_string>`. All of the other mechanisms
for setting hosts are interpreted by the ``fab`` tool when it runs, and don't
matter when running as a library.

That said, most use cases where you want to marry a given task ``X`` and a given list of hosts ``Y`` can, as of Fabric 1.3, be handled with the `~fabric.tasks.execute` function via ``execute(X, hosts=Y)``. Please see `~fabric.tasks.execute`'s documentation for details -- manual host string manipulation should be rarely necessary.

Disconnecting
=============

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


Final note
==========

This document is an early draft, and may not cover absolutely every difference
between ``fab`` use and library use. However, the above should highlight the
largest stumbling blocks. When in doubt, note that in the Fabric source code,
``fabric/main.py`` contains the bulk of the extra work done by ``fab``, and may
serve as a useful reference.
