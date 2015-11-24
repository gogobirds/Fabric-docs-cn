=======
和远程程序交互
=======

Fabric的主要操作, `~fabric.operations.run` 和 `~fabric.operations.sudo`,
能够发送本地输入到远程，其方式几乎等同于 ``ssh`` 程序. 例如, 程序显示密码提示
(如. 数据库被更替, 或者修改了用户密码) 的行为就想你直接与它们交互.

然而, 与 ``ssh`` 对比, Fabric的实现功能是有一些局限而不总是直观的. 本文档将讨论这些细节.

.. note::
    不熟悉Unix标准输出, 错误输出管道和(或)终端设备的读者，可以分别访问维基页面`Unix pipelines
    <http://en.wikipedia.org/wiki/Pipe_(Unix)>`_ 和 `Pseudo terminals
    <http://en.wikipedia.org/wiki/Pseudo_terminal>`_ .


.. _combine_streams:

结合stdout 和 stderr
=============

第一个要注意到的问题是为什么要区分 stdout 和 stderr 流, 或在需要是结合.

缓冲区
---

在Fabric 0.9.x 之前, 和Python本身, 缓冲区在一行接一行的基础上输出:
文本不会被打印给用户直到一个换行符被发现. 在大部分情况下工作得很好但是当人们需要
处理多行输出如提示时会变成问题.

.. note::
    基于行缓冲的输出可以使程序出现暂停或无原因的暂停, 作为提示答应出来文本却不换行,
    等待用户的输入并键入回车.

新版本的Fabric缓冲区的输入输出基于字符是为了使提示带有交互作用. 这样的交互作用的可以使
复杂的程序能够更方便使用，利用 "curses" 库或以其他方式绘制屏幕 (考虑 ``top``).

利用流
---

不幸的是，同时输出stderr和stdout (正如许多程序所做的那样) 意味着两个输出流同时独立地打印
一个字节, 这样在一起会变得混乱. 而这时可以通过行缓存减轻问题，但仍然是一个严重的问题.

为了解决这个问题, Fabric使用一个设置在我们的SSH层合并两个流在低等级使得输出显得更自然.
该设置代表着 :ref:`combine-stderr` 环境变量和关键字参数，默认为 ``True``.

由于这个默认设置，输出将会显示正确，但是在 `~fabric.operations.run`/`~fabric.operations.sudo`
的返回值的 ``.stderr``  属性上为空，因为所有的输出都被定向到stdout.

反过来, 用户需要不同的 stderr 流的在Python层面和不被面向用户的输出的干扰
(或隐藏stdout和stderr从命令行) 可能会根据需要选择设置为 ``False``.


.. _pseudottys:

伪终端
===

另一个主要问题是给出交互式提示给用户时考虑如何输出用户自己的输入.

输出反馈
----

典型的终端应用或者富文本终端 (例如使用没有运行GUI的Unix系统) 等储蓄与终端驱动叫做
tty或pty (伪终端). 这些自动输出所有文本给用户 (通过stdout), 因为没有交互所以看到你
刚刚的输入是困难的. 终端设别也可以有条件的的关闭响应，允许安全的密码提示.

然而，它可能是在没有一个tty或pty的下运行的程序(例如cron任务) 在这种情况下，任何stdin
数据在输入后都不会被回显.这是在没有人的周围使所期望的的方案，这是Fabric旧版本默认的操作
模式.

Fabric的方法
---------

不幸的事在, 在通过Fabric执行命令的上下文中, 当没有pty的存在来呼应用户输入,
Fabric必须的自己响应它们. 这足够应用于很多应用, 但是它对密码输入有反馈，变得不安全.

在满足安全利益的原则下 (因为它们运行在终端模拟器下，只要用户期望该行为发生), Fabric 1.0
往后的版本默认在pty下运行. 随着pty的开启, Fabric 简单的允许远程处理输出和隐藏sdtin
而不反馈任何东西.

.. note::
    除了允许正常的反馈行为，一个pty也意味着当程序连接到终端将会有不同的行为. 例如,
    多颜色程序在终端输出但是运行在后态势不会打印有颜色的输出. 警惕这一点如果你期望
    `~fabric.operations.run` 或者 `~fabric.operations.sudo` 的返回值.

对于需要将pty关闭的情况, 可以使用 :option:`--no-pty` 命令行选项和 :ref:`always-use-pty`
环境变量.

结合两者
====

最后一点，请记住使用伪终端可以有效地结合stdout和stderr -- 有一个大致相同的方法
:ref:`combine_stderr <combine_streams>`. 这是因为终端设备能够自然地发送stdout和stderr
到想同的地方 -- 用户的显示 -- 这样使得难以区分它们.

然而, 在Fabric级别, 两组彼此不同的设置可以有多种方式组合. 默认值都设置为``True``;
其他的组合如下:

* ``run("cmd", pty=False, combine_stderr=True)``: 将会使Fabric输出所有的stdin, 包括密码,
  以及可能改变``cmd``的行为. 如果 ``cmd`` 的行为不希望在pty下运行时很有用，你不用担心密码提示.

* ``run("cmd", pty=False, combine_stderr=False)``: 都设置为``False``, Fabric会输出
  stdin但不会引发pty的问题 -- 这极有可能导致不期望的行为除了简单的命令.
  然而, 这也是访问不同的stderr流的唯一途径，偶尔很有用.
* ``run("cmd", pty=True, combine_stderr=False)``: 有效的，但是不会造成真正的区别,
  如 ``pty=True`` 仍然会合并流.
  可能在避免任何 ``combine_stderr`` 的问题是有用 (目前已知都没有).
