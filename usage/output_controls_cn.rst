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

标准的原子输出级别/组如下:

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

消除故障时有两个可供使用的原子输出级别:
``debug``, 和剩下那个级别的行为稍有不同; ``exceptions``, 其行为包含于 ``debug`` 但是可能分别启用.

* **debug**:``debug``, 打开调试 (默认关闭). 目前, 这被用于查看当前运行的"全部"命令; 例如下面这个
  `~fabric.operations.run` 调用::

      run('ls "/home/username/Folder Name With Spaces/"')

  通常, ``running`` 一行会显示传递给 `~fabric.operations.run`的到底是什么, 比如::

      [hostname] run: ls "/home/username/Folder Name With Spaces/"

  打开 ``debug``, 假设你忘了将 :ref:`shell` 设置为 ``True``, 你会看到传递给远程服务器的完整字符串::

      [hostname] run: /bin/bash -l -c "ls \"/home/username/Folder Name With Spaces\""

  启用 ``debug`` 输出将会显示中断时完整的Python信息回溯(如果启用了 ``exceptions`` 输出).
  
  .. note::
  
      修改了其它的输出,(就像上例中,修改了'running'一行以显示shell和所有遗失的字符),
      此设置会优先于其它的设置; 因此如果 ``running`` 为False但 ``debug`` 为True,
      你仍然会看到处于调试模式的'running'行.

* **exceptions**: 发生异常时会显示回溯信息; 即使 ``debug`` 设为 ``False``, 仍会有人为了使用而对详细的错误信息感兴趣.

.. versionchanged:: 1.0
    目前调试输出包括了中断时的完整Python回溯信息.

.. versionchanged:: 1.11
    添加了 ``exceptions`` 输出级别.

.. _output-aliases:

输出级别别名
------

除了以上原子级/独立的级别, Fabric还提供了若干便捷的别名, 以映射到多个其他级别.
这些可能在引用了其他级别的任何地方引用, 并且将会有效地在它们映射的级别中切换.

* **output**: 同时映射到 ``stdout`` 和 ``stderr``. 若你只想看'running'行和自己的语句输出(以及警告), 这将会很有用.

* **everything**: 包括``warnings``, ``running``, ``user`` 和 ``output``
  (参见上文.) 因此, 关闭 ``everything``, 你会看到几乎最少的输出(如果有也只会是 ``status`` 和 ``debug`` ),
  连同你自己的语句输出.

* **commands**: 包括 ``stdout`` 和 ``running``. 不显示任何标准错误输出时, 有利于完全隐藏没有错误的命令.

.. versionchanged:: 1.4
    添加 ``commands`` 输出别名.


隐藏 和/或 显示输出级别
=============
你有很多种切换Fabric输出级别的方法; 为例, 请在每个重点处参见API文档:

* **Direct modification of fabric.state.output**: `fabric.state.output` 是一个字典子类(和 :doc:`env <env>`相似),
  其键为输出级别的名称, 值为True (以显示特定类型的输出)或False (不显示).
  
  `fabric.state.output` 是输出级别的最低级别体现, 也是决定输出与否时的Fabric的内部引用.

* **Context managers**: `~fabric.context_managers.hide` 和`~fabric.context_managers.show`
　是采取了一个或多个输出级别名作为字符串的一对孪生上下文管理器, 也通过封装块来显示或隐藏a.
　和Fabric的其它上下文管理器一样, 块退出时之前的值会被还原.

  .. seealso::

      `~fabric.context_managers.settings`, 可以嵌套调用
      `~fabric.context_managers.hide` 和/或 自身内部的`~fabric.context_managers.show`.

* **Command-line arguments**: 你可能会使用 :doc:`fab`的 :option:`--hide` 和/或
  :option:`--show` 参数, 这和上下文管理器的行为完全一致 －－ 一样的名字(但是当然在全局范围内应用)
  并且同样以逗号分隔的字符串作为输入.
