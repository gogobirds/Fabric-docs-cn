====
管理输出
====

默认的 ``fab``工具非常冗余, 会输出几乎所有它包含的东西, 包括远程终端的标准错误和标准输出流等等.
虽然很多情况下知道在进行什么是很有必要的, 任何重要的Fabric任务运行时都很难被追踪.


输出级别
====

为了便于组织输出, Fabric输出被分为许多重叠的级别或组, 每个级别或组都可以独立地开启或关闭.
这样为用户所见的输出提供了灵活的控制.

.. note::

    所有级别, 除了 ``debug`` 和 ``exceptions``, 都默认启用.

标准输出级别
------

标准的atomic输出级别/组如下:

* **status**: 状态信息, 如果用户使用键入中断,或者当服务器断开连接时, Fabric完成运行时记录, .
这些信息几乎都是相关的, 但不详细.

* **aborts**: 中断的消息. 正如状态信息, 将Fabric作为库使用时, 这些只应该被关闭或者也可能没有.
值得注意的是, 即使该输出组被关闭, 也会出现中断 -- 不会有任何Fabric为何中断的输出信息!

* **warnings**: 警告消息. 当考虑操作可能失败时, 这些可能常被关闭,
比如使用``grep``来检测文件中文段的存在与否时. 如果同时将 ``env.warn_only`` 设为True,
远程中断失败时会导致完全无记载的警告. 正如``aborts``, 此设置不控制实际的警告行为, 仅决定警告消息输出与否.

* **running**: 命令执行或文件转移的输出, 例如: ``[myserver] run: ls /var/www``.
同时控制输出正在运行的任务, 例如:``[myserver] Executing task 'foo'``.

* **stdout**: 本地或远程的标准输出, 即命令的无错误输出.

* **stderr**: 本地或远程的标准错误输出, 即命令的错误信息输出.

* **user**: 用户生成的输出, 即fabfile代码通过使用 `~fabric.utils.fastprint` 或
 `~fabric.utils.puts` 函数打印的本地输出.

.. versionchanged:: 0.9.2
    添加 "Executing task" 行至 ``running`` 输出级别.

.. versionchanged:: 0.9.2
    添加 ``user`` 输出级别.

调试输出
----

There are two more atomic output levels for use when troubleshooting:
``debug``, which behaves slightly differently from the rest, and
``exceptions``, whose behavior is included in ``debug`` but may be enabled
separately.

* **debug**: Turn on debugging (which is off by default.) Currently, this is
  largely used to view the "full" commands being run; take for example this
  `~fabric.operations.run` call::

      run('ls "/home/username/Folder Name With Spaces/"')

  Normally, the ``running`` line will show exactly what is passed into
  `~fabric.operations.run`, like so::

      [hostname] run: ls "/home/username/Folder Name With Spaces/"

  With ``debug`` on, and assuming you've left :ref:`shell` set to ``True``, you
  will see the literal, full string as passed to the remote server::

      [hostname] run: /bin/bash -l -c "ls \"/home/username/Folder Name With Spaces\""

  Enabling ``debug`` output will also display full Python tracebacks during
  aborts (as if ``exceptions`` output was enabled).
  
  .. note::
  
      Where modifying other pieces of output (such as in the above example
      where it modifies the 'running' line to show the shell and any escape
      characters), this setting takes precedence over the others; so if
      ``running`` is False but ``debug`` is True, you will still be shown the
      'running' line in its debugging form.

* **exceptions**: Enables display of tracebacks when exceptions occur; intended
  for use when ``debug`` is set to ``False`` but one is still interested in
  detailed error info.

.. versionchanged:: 1.0
    Debug output now includes full Python tracebacks during aborts.

.. versionchanged:: 1.11
    Added the ``exceptions`` output level.

.. _output-aliases:

Output level aliases
--------------------

In addition to the atomic/standalone levels above, Fabric also provides a
couple of convenience aliases which map to multiple other levels. These may be
referenced anywhere the other levels are referenced, and will effectively
toggle all of the levels they are mapped to.

* **output**: Maps to both ``stdout`` and ``stderr``. Useful for when you only
  care to see the 'running' lines and your own print statements (and warnings).

* **everything**: Includes ``warnings``, ``running``, ``user`` and ``output``
  (see above.) Thus, when turning off ``everything``, you will only see a bare
  minimum of output (just ``status`` and ``debug`` if it's on), along with your
  own print statements.

* **commands**: Includes ``stdout`` and ``running``. Good for hiding
  non-erroring commands entirely, while still displaying any stderr output.

.. versionchanged:: 1.4
    Added the ``commands`` output alias.


Hiding and/or showing output levels
===================================

You may toggle any of Fabric's output levels in a number of ways; for examples,
please see the API docs linked in each bullet point:

* **Direct modification of fabric.state.output**: `fabric.state.output` is a
  dictionary subclass (similar to :doc:`env <env>`) whose keys are the output
  level names, and whose values are either True (show that particular type of
  output) or False (hide it.)
  
  `fabric.state.output` is the lowest-level implementation of output levels and
  is what Fabric's internals reference when deciding whether or not to print
  their output.

* **Context managers**: `~fabric.context_managers.hide` and
  `~fabric.context_managers.show` are twin context managers that take one or
  more output level names as strings, and either hide or show them within the
  wrapped block. As with Fabric's other context managers, the prior values are
  restored when the block exits.

  .. seealso::

      `~fabric.context_managers.settings`, which can nest calls to
      `~fabric.context_managers.hide` and/or `~fabric.context_managers.show`
      inside itself.

* **Command-line arguments**: You may use the :option:`--hide` and/or
  :option:`--show` arguments to :doc:`fab`, which behave exactly like the
  context managers of the same names (but are, naturally, globally applied) and
  take comma-separated strings as input.
