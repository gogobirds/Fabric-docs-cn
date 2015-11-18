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

Filename pattern which ``fab`` searches for when loading fabfiles.
To indicate a specific file, use the full path to the file. Obviously, it
doesn't make sense to set this in a fabfile, but it may be specified in a
``.fabricrc`` file or on the command line.

.. seealso:: :option:`--fabfile <-f>`, :doc:`fab`


.. _gateway:

``gateway``
-----------

**默认值:** ``None``

Enables SSH-driven gatewaying through the indicated host. The value should be a
normal Fabric host string as used in e.g. :ref:`env.host_string <host_string>`.
When this is set, newly created connections will be set to route their SSH
traffic through the remote SSH daemon to the final destination.

.. versionadded:: 1.5

.. seealso:: :option:`--gateway <-g>`


.. _host_string:

``host_string``
---------------

**默认值:** ``None``

Defines the current user/host/port which Fabric will connect to when executing
`~fabric.operations.run`, `~fabric.operations.put` and so forth. This is set by
``fab`` when iterating over a previously set host list, and may also be
manually set when using Fabric as a library.

.. seealso:: :doc:`execution`


.. _forward-agent:

``forward_agent``
--------------------

**默认值:** ``False``

If ``True``, enables forwarding of your local SSH agent to the remote end.

.. versionadded:: 1.4

.. seealso:: :option:`--forward-agent <-A>`

.. _host:

``host``
--------

**默认值:** ``None``

Set to the hostname part of ``env.host_string`` by ``fab``. For informational
purposes only.

.. _hosts:

``hosts``
---------

**默认值:** ``[]``

The global host list used when composing per-task host lists.

.. seealso:: :option:`--hosts <-H>`, :doc:`execution`

.. _keepalive:

``keepalive``
-------------

**默认值:** ``0`` (i.e. no keepalive)

An integer specifying an SSH keepalive interval to use; basically maps to the
SSH config option ``ServerAliveInterval``. Useful if you find connections are
timing out due to meddlesome network hardware or what have you.

.. seealso:: :option:`--keepalive`
.. versionadded:: 1.1


.. _key:

``key``
----------------

**默认值:** ``None``

A string, or file-like object, containing an SSH key; used during connection
authentication.

.. note::
    The most common method for using SSH keys is to set :ref:`key-filename`.

.. versionadded:: 1.7


.. _key-filename:

``key_filename``
----------------

**默认值:** ``None``

May be a string or list of strings, referencing file paths to SSH key files to
try when connecting. Passed through directly to the SSH layer. May be
set/appended to with :option:`-i`.

.. seealso:: `Paramiko's documentation for SSHClient.connect() <http://docs.paramiko.org/en/latest/api/client.html#paramiko.client.SSHClient.connect>`_

.. _env-linewise:

``linewise``
------------

**默认值:** ``False``

Forces buffering by line instead of by character/byte, typically when running
in parallel mode. May be activated via :option:`--linewise`. This option is
implied by :ref:`env.parallel <env-parallel>` -- even if ``linewise`` is False,
if ``parallel`` is True then linewise behavior will occur.

.. seealso:: :ref:`linewise-output`

.. versionadded:: 1.3


.. _local-user:

``local_user``
--------------

A read-only value containing the local system username. This is the same value
as :ref:`user`'s initial value, but whereas :ref:`user` may be altered by CLI
arguments, Python code or specific host strings, :ref:`local-user` will always
contain the same value.

.. _no_agent:

``no_agent``
------------

**默认值:** ``False``

If ``True``, will tell the SSH layer not to seek out running SSH agents when
using key-based authentication.

.. versionadded:: 0.9.1
.. seealso:: :option:`--no_agent <-a>`

.. _no_keys:

``no_keys``
------------------

**默认值:** ``False``

If ``True``, will tell the SSH layer not to load any private key files from
one's ``$HOME/.ssh/`` folder. (Key files explicitly loaded via ``fab -i`` will
still be used, of course.)

.. versionadded:: 0.9.1
.. seealso:: :option:`-k`

.. _env-parallel:

``parallel``
-------------------

**默认值:** ``False``

When ``True``, forces all tasks to run in parallel. Implies :ref:`env.linewise
<env-linewise>`.

.. versionadded:: 1.3
.. seealso:: :option:`--parallel <-P>`, :doc:`parallel`

.. _password:

``password``
------------

**默认值:** ``None``

The 默认值 password used by the SSH layer when connecting to remote hosts,
**and/or** when answering `~fabric.operations.sudo` prompts.

.. seealso:: :option:`--initial-password-prompt <-I>`, :ref:`env.passwords <passwords>`, :ref:`password-management`

.. _passwords:

``passwords``
-------------

**默认值:** ``{}``

This dictionary is largely for internal use, and is filled automatically as a
per-host-string password cache. Keys are full :ref:`host strings
<host-strings>` and values are passwords (strings).

.. warning::
    If you modify or generate this dict manually, **you must use fully
    qualified host strings** with user and port values. See the link above for
    details on the host string API.

.. seealso:: :ref:`password-management`

.. versionadded:: 1.0


.. _env-path:

``path``
--------

**默认值:** ``''``

Used to set the ``$PATH`` shell environment variable when executing commands in
`~fabric.operations.run`/`~fabric.operations.sudo`/`~fabric.operations.local`.
It is recommended to use the `~fabric.context_managers.path` context manager
for managing this value instead of setting it directly.

.. versionadded:: 1.0


.. _pool-size:

``pool_size``
-------------

**默认值:** ``0``

Sets the number of concurrent processes to use when executing tasks in parallel.

.. versionadded:: 1.3
.. seealso:: :option:`--pool-size <-z>`, :doc:`parallel`

.. _prompts:

``prompts``
-------------

**默认值:** ``{}``

The ``prompts`` dictionary allows users to control interactive prompts. If a
key in the dictionary is found in a command's standard output stream, Fabric
will automatically answer with the corresponding dictionary value.

.. versionadded:: 1.9

.. _port:

``port``
--------

**默认值:** ``None``

Set to the port part of ``env.host_string`` by ``fab`` when iterating over a
host list. May also be used to specify a 默认值 port.

.. _real-fabfile:

``real_fabfile``
----------------

**默认值:** ``None``

Set by ``fab`` with the path to the fabfile it has loaded up, if it got that
far. For informational purposes only.

.. seealso:: :doc:`fab`


.. _remote-interrupt:

``remote_interrupt``
--------------------

**默认值:** ``None``

Controls whether Ctrl-C triggers an interrupt remotely or is captured locally,
as follows:

* ``None`` (the 默认值): only `~fabric.operations.open_shell` will exhibit
  remote interrupt behavior, and
  `~fabric.operations.run`/`~fabric.operations.sudo` will capture interrupts
  locally.
* ``False``: even `~fabric.operations.open_shell` captures locally.
* ``True``: all functions will send the interrupt to the remote end.

.. versionadded:: 1.6


.. _rcfile:

``rcfile``
----------

**默认值:** ``$HOME/.fabricrc``

Path used when loading Fabric's local settings file.

.. seealso:: :option:`--config <-c>`, :doc:`fab`

.. _reject-unknown-hosts:

``reject_unknown_hosts``
------------------------

**默认值:** ``False``

If ``True``, the SSH layer will raise an exception when connecting to hosts not
listed in the user's known-hosts file.

.. seealso:: :option:`--reject-unknown-hosts <-r>`, :doc:`ssh`

.. _system-known-hosts:

``system_known_hosts``
------------------------

**默认值:** ``None``

If set, should be the path to a :file:`known_hosts` file.  The SSH layer will
read this file before reading the user's known-hosts file.

.. seealso:: :doc:`ssh`

.. _roledefs:

``roledefs``
------------

**默认值:** ``{}``

Dictionary defining role name to host list mappings.

.. seealso:: :doc:`execution`

.. _roles:

``roles``
---------

**默认值:** ``[]``

The global role list used when composing per-task host lists.

.. seealso:: :option:`--roles <-R>`, :doc:`execution`

.. _shell:

``shell``
---------

**默认值:** ``/bin/bash -l -c``

Value used as shell wrapper when executing commands with e.g.
`~fabric.operations.run`. Must be able to exist in the form ``<env.shell>
"<command goes here>"`` -- e.g. the 默认值 uses Bash's ``-c`` option which
takes a command string as its value.

.. seealso:: :option:`--shell <-s>`,
             :ref:`FAQ on bash as 默认值 shell <faq-bash>`, :doc:`execution`

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
