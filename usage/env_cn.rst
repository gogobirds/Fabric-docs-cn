=============
环境字典, ``env``
=============
Fabric一个简单却又完整的概念被认为是“环境” :
    一个python的字典子类,被用作是组合设置注册表和内部任务的共享数据空间.

环境字典被视为是全局变量,``fabric.state.env``为使用方便也内嵌在``fabric.api``.
``env``关键字有时也被称为"env variables".

环境配置
====
Fabric的大部分操作都是通过修改``env``变量来控制, 比如``env.hosts``(参见 :ref:`the tutorial <defining-connections>`).
其他常进行修改的变量如下：

* ``user``: 进行SSH连接时,Fabric默认使用本地用户名,如果有必要可以使用``env.user``覆盖此变量.
    至于如何在每个主机上设置用户名,参见 :doc:`execution`.
* ``password``: 用于显示设置默认连接或者sudo密码.假若此变量未设置或无效,Fabric会提醒你.
* ``warn_only``: 当远程终端检测到错误时,决定Fabric是否退出的布尔变量. 更多信息参见 :doc:`execution`.

更多其他环境变量的完整列表,参见本文档末尾.

`~fabric.context_managers.settings` 上下文管理器
---

很多时候,为便于给定的设置变动只针对一个代码块,暂时修改``env``变得很有用.
Fabric提供`~fabric.context_managers.settings`上下文管理器,接受任意多个"键/值"对,
并将其用于修改封装好的代码块中的``env``.

例如,很多情形下设置 ``warn_only`` 都很有用（如下）.
将其提供给少数几行代码,使用 ``settings(warn_only=True)``,
参见简化版的 ``contrib`` `~fabric.contrib.files.exists` 函数::

    from fabric.api import settings, run

    def exists(path):
        with settings(warn_only=True):
            return run('test -e %s' % path)

关于上下文管理器和其他类似的工具,请参见 :doc:`../api/core/context_managers` API文档.

环境共享状态
======

已经提到过,``env`` 对象纯粹是个字典子类,所以你自己的fabfile代码也可以存储信息.
有时在多个任务间保持状态但只运行一个操作,这就变得很有用了.

.. note::

    ``env``这个特性是历史性的：在以前,fabfiles不是纯python编写,因此环境成了任务间交互信息的唯一路径.
    如今,你可以直接调用其他任务或子例程,甚至在需要时保存模块级别共享状态.

    在以后的版本里,Fabric的线程会更安全。
    对于这一点, ``env``可能会成为保持全局状态的唯一简便、安全途径.

其他考虑
====

在继承``dict``时,Fabric的 ``env`` 也已作修改,以便于它的值可以通过属性访问来读写,正如上文所属.
换句话说,``env.host_string``和``env['host_string']``的作用相同.
通常我们会发现：属性访问的方式能减少敲键盘的工作量,并提高代码的可读性.
因此这也是和``env``交互的推荐方式.

实际上字典在其他用途中也很有用,比如Python基于 ``dict``的字符串替代,当你需要在一个字符串中插入多个环境变量时就很便捷了.
使用"普通"字符串替代就可能像下面这样::

    print("Executing on %s as %s" % (env.host, env.user))

使用字典风格的替代就更可读并简短::

    print("Executing on %(host)s as %(user)s" % env)

.. _env-vars:

环境变量的完整列表
=========

以下是所有预定义（或者Fabric运行时自定义）的环境变量.
它们中大多数被直接操作时,一般最好使用 ~fabric.context_managers`,
或者`~fabric.context_managers.settings`或者特定的上下文管理器,比如`~fabric.context_managers.cd`.

需要注意的是它们中的很多都可以通过``fab``的命令行参数来设置,参见:doc:`fab`.
适当的地方提供有交叉引用【Cross-references】.

.. seealso:: :option:`--set`

.. _abort-exception:

``abort_exception``
-------------------

**默认值:** ``None``

Fabric 处理中断时，通常将错误信息反馈给标准错误输出,并且调用 ``sys.exit(1)``.
此设置允许对操作进行覆写(当 ``env.abort_exception`` 为 ``None``时).

赋给一个可调用的对象,它可以接受一个字符串(反馈的错误信息)并返回一个异常实例.
实例对象会被抛出,而不是通过( ``sys.exit`` 执行的) ``SystemExit`` .

很多时候你会想要简单地设置一个异常类, 它完美地符合以上描述 (可调用,接受字符串,返回异常实例).
例如： ``env.abort_exception = MyExceptionClass``.

.. _abort-on-prompts:

``abort_on_prompts``
--------------------

**默认值:** ``False``

当值为 ``True``, Fabric会以无交互模式运行,任何时候调用
`~fabric.utils.abort` ,它都会提示用户进行输入 (比如:提示输入密码,询问连接到哪台主机,
fabfile触发`~fabric.operations.prompt`, 诸如此类.) 这使得用户可以确保Fabric的会话能清楚地中止,
而不是当发生不可预料的情况时,一直处于等待用户输入的界面.

.. versionadded:: 1.1
.. seealso:: :option:`--abort-on-prompts`


``all_hosts``
-------------

**默认值:** ``[]``

 ``fab`` 设置的正在执行命令的完整主机列表.仅供显示信息.

.. seealso:: :doc:`execution`

.. _always-use-pty:

``always_use_pty``
------------------

**默认值:** ``True``

设置为 ``False``时, 使 `~fabric.operations.run`/`~fabric.operations.sudo`
的行为就像它们被 ``pty=False``调用一样.

.. seealso:: :option:`--no-pty`
.. versionadded:: 1.0

.. _colorize-errors:

``colorize_errors``
-------------------

**默认值** ``False``

设置为 ``True``时,终端输出的错误信息会显示成红色,警告则显示为洋红色,以便更容易被看见.

.. versionadded:: 1.7

.. _combine-stderr:

``combine_stderr``
------------------

**默认值**: ``True``

使SSH层合并远程程序的stdout和stderr流,以避免输出的时候混合在一起.
为何需要这个功能以及效果如何,请参见:ref:`combine_streams`.

.. versionadded:: 1.0

``command``
-----------

**默认值:** ``None``

由``fab``为当前执行命令设置的名称,(例如,当执行 ``$ fab task1 task2``时,
``env.command``会在执行 ``task1``时被设置为 ``"task1"``,之后又被设置为 ``"task2"``).
仅供显示信息.

.. seealso:: :doc:`execution`

``command_prefixes``
--------------------

**默认值:** ``[]``

由 `~fabric.context_managers.prefix`修改,并附加在由
 `~fabric.operations.run`/`~fabric.operations.sudo`执行的命令前面.

.. versionadded:: 1.0

.. _command-timeout:

``command_timeout``
-------------------

**默认值:** ``None``

远程命令的超时时间,单位为秒.

.. versionadded:: 1.6
.. seealso:: :option:`--command-timeout`

.. _connection-attempts:

``connection_attempts``
-----------------------

**默认值:** ``1``

当连接到一台新服务器时,Fabric尝试重新连接的次数.由于向后兼容的原因,默认为一次连接.

.. versionadded:: 1.4
.. seealso:: :option:`--connection-attempts`, :ref:`timeout`

``cwd``
-------

**默认值:** ``''``

当前工作目录.用于为 `~fabric.context_managers.cd`上下文管理器保持状态.

.. _dedupe_hosts:

``dedupe_hosts``
----------------

**默认值:** ``True``

合并主机列表时去除重复项,以使给出的主机串都只出现一次
(例如,当使用 ``@hosts`` + ``@roles``或 ``-H`` + ``-R``的组合时).

设置为 ``False``时,此操作不会去除重复项,使得用户能显示地在某个主机上多次运行一个任务
(换个说法,以并行或串行的方式).

.. versionadded:: 1.5

.. _disable-known-hosts:

``disable_known_hosts``
-----------------------

**默认值:** ``False``

If ``True``, the SSH layer will skip loading the user's known-hosts file.
Useful for avoiding exceptions in situations where a "known host" changing its
host key is actually valid (e.g. cloud servers such as EC2.)

.. seealso:: :option:`--disable-known-hosts <-D>`, :doc:`ssh`


.. _eagerly-disconnect:

``eagerly_disconnect``
----------------------

**默认值:** ``False``

为 ``True``时,会使 ``fab``在每个独立的任务执行完后关闭连接,
而不是在整个运行完成之后.这会防止很多无用的网络会话大量堆积,
或者防止每个进程打开的文件、网络硬件的因限制引起的问题.

.. note::
    当打开时,此设置将会使断开连接的信息贯穿在所有输出信息中,
    而不是只在结束的末尾.在未来版本中可能会改进.

.. _effective_roles:

``effective_roles``
-------------------

**默认值:** ``[]``

由 ``fab``设置的当前执行命令的角色列表.仅供信息显示.

.. versionadded:: 1.9
.. seealso:: :doc:`execution`

.. _exclude-hosts:

``exclude_hosts``
-----------------

**默认值:** ``[]``

指定一个主机串列表为 ``fab`` 执行中的 :ref:`skipped over <exclude-hosts>`.
通过 :option:`--exclude-hosts/-x <-x>`设置.

.. versionadded:: 1.1


``fabfile``
-----------

**默认值:** ``fabfile.py``

加载fabfiles时, ``fab``查找的文件名0.
为了指定一个特定的文件名,使用此文件的完整路径.显然, 在fabfile里这样设置并无意义,
但是它可以在 ``.fabricrc``文件和命令行里进行设置.

.. seealso:: :option:`--fabfile <-f>`, :doc:`fab`


.. _gateway:

``gateway``
-----------

**默认值:** ``None``

可以通过指定主机创建SSH驱动的网关.它的值应该是普通的Fabric主机串,
n就像在 :ref:`env.host_string <host_string>`中使用的一样.
设置时,新建的连接将通过远程SSH的守护进程连接到目的终点.

.. versionadded:: 1.5

.. seealso:: :option:`--gateway <-g>`


.. _host_string:

``host_string``
---------------

**默认值:** ``None``

定义执行 `~fabric.operations.run`/ `~fabric.operations.put` 等命令时,Fabric将要连接到的用户/主机/端口.
由 ``fab``与以前设置的主机列表交互时设置,将Fabric当作库使用时也可能进行手动设置.

.. seealso:: :doc:`execution`


.. _forward-agent:

``forward_agent``
-----------------

**默认值:** ``False``

为 ``True``,则能使本地的SSH代理转发给远程终端.

.. versionadded:: 1.4

.. seealso:: :option:`--forward-agent <-A>`

.. _host:

``host``
--------

**默认值:** ``None``

由``fab``为 ``env.host_string``的主机名块设置.仅供信息显示.

.. _hosts:

``hosts``
---------

**默认值:** ``[]``

创建每个任务的主机列表时的全局主机列表.

.. seealso:: :option:`--hosts <-H>`, :doc:`execution`

.. _keepalive:

``keepalive``
-------------

**默认值:** ``0`` (i.e. no keepalive)

一个用于指定SSH维持时间的整数; 主要映射到SSH配置选项的 ``ServerAliveInterval``.
当碍事的网络硬件或其他原因造成连接超时时,该变量将派上用场.

.. seealso:: :option:`--keepalive`
.. versionadded:: 1.1


.. _key:

``key``
----------------

**默认值:** ``None``

一个字符串或文件类对象,包括了SSH秘钥;连接验证时使用.

.. note::
    使用SSH秘钥最常用的方式即为设置变量 :ref:`key-filename`.

.. versionadded:: 1.7


.. _key-filename:

``key_filename``
----------------

**默认值:** ``None``

字符串或字符串列表, 连接时,SSH秘钥文件尝试的引入文件路径. 直接通过SSH层.
可以通过 :option:`-i`设置/添加.

.. seealso:: `Paramiko关于SSHClient.connect()的文档
<http://docs.paramiko.org/en/latest/api/client.html#paramiko.client.SSHClient.connect>`_

.. _env-linewise:

``linewise``
------------

**默认值:** ``False``
以行为单位进行缓冲,而不是字符/字节,通常在并行模式下运行时.
可以通过 :option:`--linewise`设置.该操作隐含于 :ref:`env.parallel <env-parallel>`
 -- 即使 ``linewise``设置为False,若 ``parallel``为True,linewise行为仍会存在.

.. seealso:: :ref:`linewise-output`

.. versionadded:: 1.3


.. _local-user:

``local_user``
--------------

一个包含本地系统用户名的只读值.与:ref:`user`的初始值为相同值,
但是 :ref:`user`可以被CLI参数值、Python代码或指定主机串改动,
:ref:`local-user`总是包含相同值.

.. _no_agent:

``no_agent``
------------

**默认值:** ``False``

值为 ``True``, 则SSH层在使用基于秘钥的验证时,不会查找运行SSH代理.

.. versionadded:: 0.9.1
.. seealso:: :option:`--no_agent <-a>`

.. _no_keys:

``no_keys``
------------------

**默认值:** ``False``

值为``True``,SSH层不会从个人的 ``$HOME/.ssh/``文件加载任何私钥文件.
 (当然,通过 ``fab -i``显示加载的秘钥文件仍能使用.)

.. versionadded:: 0.9.1
.. seealso:: :option:`-k`

.. _env-parallel:

``parallel``
-------------------

**默认值:** ``False``

值为``True``,会使所有任务以并行模式运行. 参见 :ref:`env.linewise
<env-linewise>`.

.. versionadded:: 1.3
.. seealso:: :option:`--parallel <-P>`, :doc:`parallel`

.. _password:

``password``
------------

**默认值:** ``None``

连接远程主机时,并/或回应 `~fabric.operations.sudo`提示时,SSH层使用的默认密码.

.. seealso:: :option:`--initial-password-prompt <-I>`, :ref:`env.passwords <passwords>`, :ref:`password-management`

.. _passwords:

``passwords``
-------------

**默认值:** ``{}``

这个字典主要供内部使用,被自动填充每个主机串的密码缓存.
键是完整的 :ref:`host strings
<host-strings>` ,值为密码(字符串).

.. warning::
    如果你修改或手动生成这个字典,**你必须使用完整合格的主机串**以及用户和端口值.
    关于主机串API的更多信息,参见以下链接.

.. seealso:: :ref:`password-management`

.. versionadded:: 1.0


.. _env-path:

``path``
--------

**默认值:** ``''``

在`~fabric.operations.run`/`~fabric.operations.sudo`/`~fabric.operations.local`执行命令时,
用于设置 ``$PATH`` shell环境变量.
管理此变量值需要用 `~fabric.context_managers.path`上下文管理器,而不是直接设置.

.. versionadded:: 1.0


.. _pool-size:

``pool_size``
-------------

**默认值:** ``0``

并发模式下执行任务时,设置并发进程的使用序号.

.. versionadded:: 1.3
.. seealso:: :option:`--pool-size <-z>`, :doc:`parallel`

.. _prompts:

``prompts``
-------------

**默认值:** ``{}``

 ``prompts``字典允许用户控制交互式提示.如果在一个命令的标准输出流中找到字典里的某个值,
 Fabric会自动回复相应的字典值.

.. versionadded:: 1.9

.. _port:

``port``
--------

**默认值:** ``None``

遍历主机列表时,由 ``fab`` 设置的 ``env.host_string``端口部分.
可以用来指定一个默认端口.

.. _real-fabfile:

``real_fabfile``
----------------

**默认值:** ``None``

 ``fab``设置的带有已加载fabfile的路径,前提是它已获得该路径.
仅供显示信息.

.. seealso:: :doc:`fab`


.. _remote-interrupt:

``remote_interrupt``
--------------------

**默认值:** ``None``

控制Ctrl-C是触发远程中断还是本地捕获,如下:

* ``None`` (默认值): 只有 `~fabric.operations.open_shell`会表现出远程中断行为,
 `~fabric.operations.run`/`~fabric.operations.sudo`则会在本地捕获中断.
* ``False``: `~fabric.operations.open_shell`本地捕获.
* ``True``: 所有函数都将中断行为发送至远程终端.

.. versionadded:: 1.6


.. _rcfile:

``rcfile``
----------

**默认值:** ``$HOME/.fabricrc``

加载Fabric本地设置文件的路径.

.. seealso:: :option:`--config <-c>`, :doc:`fab`

.. _reject-unknown-hosts:

``reject_unknown_hosts``
------------------------

**默认值:** ``False``

值为 ``True``,那么连接到未列在用户已知主机文件中的主机时，SSH层将抛出异常.

.. seealso:: :操作:`--reject-unknown-hosts <-r>`, :文档:`ssh`

.. _system-known-hosts:

``system_known_hosts``
------------------------

**默认值:** ``None``

如果已设置, 应该为 :file:`known_hosts` 文件的路径.读取用户的已知主机文件前,SSH层会先读取此文件.

.. seealso:: :doc:`ssh`

.. _roledefs:

``roledefs``
------------

**默认值:** ``{}``

为主机列表映射定义的字典对象名.

.. seealso:: :doc:`execution`

.. _roles:

``roles``
---------

**默认值:** ``[]``

构建每个任务的主机列表时,使用的全局对象列表.

.. seealso:: :option:`--roles <-R>`, :doc:`execution`

.. _shell:

``shell``
---------

**默认值:** ``/bin/bash -l -c``

Shell封装器在执行命令(例如: `~fabric.operations.run`)时使用的值.
必须存在于表格 ``<env.shell>"<command goes here>"``中-- 例如: 默认值使用Bash的 ``-c``操作,必须使用命令串作为值.

.. seealso:: :option:`--shell <-s>`,
             :ref:`FAQ on bash as default shell <faq-bash>`, :doc:`execution`

.. _skip-bad-hosts:

``skip_bad_hosts``
------------------

**默认值:** ``False``

If ``True``, causes ``fab`` (or non-``fab`` use of `~fabric.tasks.execute`) to skip over hosts it can't connect to.

.. versionadded:: 1.4
.. seealso::
    :option:`--skip-bad-hosts`, :ref:`excluding-hosts`, :doc:`execution`


.. _skip-unknown-tasks:

``skip_unknown_tasks``
----------------------

**默认值:** ``False``

If ``True``, causes ``fab`` (or non-``fab`` use of `~fabric.tasks.execute`)
to skip over tasks not found, without aborting.

.. seealso::
    :option:`--skip-unknown-tasks`


.. _ssh-config-path:

``ssh_config_path``
-------------------

**默认值:** ``$HOME/.ssh/config``

Allows specification of an alternate SSH configuration file path.

.. versionadded:: 1.4
.. seealso:: :option:`--ssh-config-path`, :ref:`ssh-config`

``ok_ret_codes``
------------------------

**默认值:** ``[0]``

Return codes in this list are used to determine whether calls to
`~fabric.operations.run`/`~fabric.operations.sudo`/`~fabric.operations.sudo`
are considered successful.

.. versionadded:: 1.6

.. _sudo_prefix:

``sudo_prefix``
---------------

**默认值:** ``"sudo -S -p '%(sudo_prompt)s' " % env``

The actual ``sudo`` command prefixed onto `~fabric.operations.sudo` calls'
command strings. Users who do not have ``sudo`` on their 默认值 remote
``$PATH``, or who need to make other changes (such as removing the ``-p`` when
passwordless sudo is in effect) may find changing this useful.

.. seealso::

    The `~fabric.operations.sudo` operation; :ref:`env.sudo_prompt
    <sudo_prompt>`

.. _sudo_prompt:

``sudo_prompt``
---------------

**默认值:** ``"sudo password:"``

Passed to the ``sudo`` program on remote systems so that Fabric may correctly
identify its password prompt.

.. seealso::

    The `~fabric.operations.sudo` operation; :ref:`env.sudo_prefix
    <sudo_prefix>`

.. _sudo_user:

``sudo_user``
-------------

**默认值:** ``None``

Used as a fallback value for `~fabric.operations.sudo`'s ``user`` argument if
none is given. Useful in combination with `~fabric.context_managers.settings`.

.. seealso:: `~fabric.operations.sudo`

.. _env-tasks:

``tasks``
-------------

**默认值:** ``[]``

Set by ``fab`` to the full tasks list to be executed for the currently
executing command. For informational purposes only.

.. seealso:: :doc:`execution`

.. _timeout:

``timeout``
-----------

**默认值:** ``10``

Network connection timeout, in seconds.

.. versionadded:: 1.4
.. seealso:: :option:`--timeout`, :ref:`connection-attempts`

``use_shell``
-------------

**默认值:** ``True``

Global setting which acts like the ``shell`` argument to
`~fabric.operations.run`/`~fabric.operations.sudo`: if it is set to ``False``,
operations will not wrap executed commands in ``env.shell``.


.. _use-ssh-config:

``use_ssh_config``
------------------

**默认值:** ``False``

Set to ``True`` to cause Fabric to load your local SSH config file.

.. versionadded:: 1.4
.. seealso:: :ref:`ssh-config`


.. _user:

``user``
--------

**默认值:** User's local username

The username used by the SSH layer when connecting to remote hosts. May be set
globally, and will be used when not otherwise explicitly set in host strings.
However, when explicitly given in such a manner, this variable will be
temporarily overwritten with the current value -- i.e. it will always display
the user currently being connected as.

To illustrate this, a fabfile::

    from fabric.api import env, run

    env.user = 'implicit_user'
    env.hosts = ['host1', 'explicit_user@host2', 'host3']

    def print_user():
        with hide('running'):
            run('echo "%(user)s"' % env)

and its use::

    $ fab print_user

    [host1] out: implicit_user
    [explicit_user@host2] out: explicit_user
    [host3] out: implicit_user

    Done.
    Disconnecting from host1... done.
    Disconnecting from host2... done.
    Disconnecting from host3... done.

As you can see, during execution on ``host2``, ``env.user`` was set to
``"explicit_user"``, but was restored to its previous value
(``"implicit_user"``) afterwards.

.. note::

    ``env.user`` is currently somewhat confusing (it's used for configuration
    **and** informational purposes) so expect this to change in the future --
    the informational aspect will likely be broken out into a separate env
    variable.

.. seealso:: :doc:`execution`, :option:`--user <-u>`

``version``
-----------

**默认值:** current Fabric version string

Mostly for informational purposes. Modification is not recommended, but
probably won't break anything either.

.. seealso:: :option:`--version <-V>`

.. _warn_only:

``warn_only``
-------------

**默认值:** ``False``

Specifies whether or not to warn, instead of abort, when
`~fabric.operations.run`/`~fabric.operations.sudo`/`~fabric.operations.local`
encounter error conditions.

.. seealso:: :option:`--warn-only <-w>`, :doc:`execution`
