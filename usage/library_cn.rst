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
其他的操作在连接 :ref:`env.host_string <host_string>` 时会被替代. 所有其他为了设定主机的机制
都由 ``fab`` 工具来运行, 当作为一个库来运行也没有关系.

也就是说, 最常见的用法是你想要给定一个 ``X`` 任务和一个 ``Y`` 主机列表, 在Fabric 1.3,
可以通过 ``execute(X, hosts=Y)`` 执行 `~fabric.tasks.execute` 函数.
请查看 `~fabric.tasks.execute` 相关文档的细节 -- 手动操作主机列表应该很少需要.

关闭连接
====

其他的关键部分,``fab`` 会在一个会话结束时为你断开所有主机的连接; 否则, Python会在那里
永远等待这些网络资源被释放.

Fabric 0.9.4 以及更新的版本你可以更容易的使用 `~fabric.network.disconnect_all`.
仅需要确认你的代码在最后调用它们 (通常在 ``try: finally`` 语句中包括 ``finally``
-- 以免你的代码的错误导致断线的发生!) 通常能够很好的工作.

如果你在Fabric 0.9.3或者更老的版本使用, 你可以简单的做到 (``disconnect_all`` 只是在这个逻辑
中增加了一点有趣的输出)::

    from fabric.state import connections

    for key in connections.keys():
        connections[key].close()
        del connections[key]


最后的提醒
=====

本文档只是一份初稿, 不能完全的覆盖 ``fab`` 和当作库使用方法的不同. 然而, 上面应该突出最大的不.
如有疑问, 请查看Fabric的源代码, ``fabric/main.py`` 包含了大量 ``fab`` 所做的工作, 作为一个有用的参考.
