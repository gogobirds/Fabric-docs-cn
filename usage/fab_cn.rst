=============
``fab`` 选项和参数
=============

通过命令行工具 ``fab`` 最通用的利用Fabfric的方式, 当Fabric安装后将会存在于你的
shell执行环境. ``fab`` 尽量成为好的Unix citizen, 使用标准的命令行风格，帮助风格等等.


基本使用
====

在最简单的形式, ``fab`` 可以没有任何选项, 跟上一个多个参数, 应该为任务名称, 例如::

    $ fab task1 task2

更多细节在 :doc:`../tutorial` 和 :doc:`execution`, 将会依次运行 ``task1``, ``task2``,
假设Farbic能够在附近找到一个包括这些任务名称的Python函数的fabfile.

然而, 扩展简单的使用到更灵活是可能的，通过使用提供的选项和(或)传递参数给个别任务.


.. _arbitrary-commands:

任意远程命令
======

.. versionadded:: 0.9.2

Fabric利用鲜为人知的命令行惯例，可以用下列方式调用::

    $ fab [options] -- [shell command]

任何跟在 ``--`` 后面的都会变成 `~fabric.operations.run` 的临时调用, 不会解析为 ``fab``
选项. 如果定义了一个主机列表在模块级别或者命令行, 这个用法相当于一个行内的匿名任务.

例如, 假设想要知道一些系统的内核版本信息; 你只需要这样做::

    $ fab -H system1,system2,system3 -- uname -a

字面上相当于下面的fabfile::

    from fabric.api import run

    def anonymous():
        run("uname -a")

由下面这样执行::

    $ fab -H system1,system2,system3 anonymous

大部分时间你想要在fabfile中写入任务 (任何使用过一次，可能会再次使用的)
但是此功能提供了一个方便快捷的方式通过SSH快速传送命令同时充分利用fabfile的连接设置.


.. _command-line-options:

命令行选项
=====

通过  ``fab --help`` 可以快速浏览所有的命令行选项. 如果想找到特定选项的细节，我们
进入细讲.

.. note::

    ``fab`` 使用Python的 `optparse`_ 库, 意味着它是典型的Linux或者GNU风格的长短选项,
    也可以自由的混合选项和参数. 例如. ``fab task1 -H hostname task2 -i path/to/keyfile``
    与 ``fab -H hostname -i path/to/keyfile task1 task2`` 等效但更直接.

.. _optparse: http://docs.python.org/library/optparse.html

.. cmdoption:: -a, --no_agent

    设置 :ref:`env.no_agent <no_agent>` 为 ``True``, 当尝试解开私钥文件时使我们的SSH
    不要连接到SSH agetn.

    .. versionadded:: 0.9.1

.. cmdoption:: -A, --forward-agent

    设置 :ref:`env.forward_agent <forward-agent>` 为 ``True``, 开启agent转发.

    .. versionadded:: 1.4

.. cmdoption:: --abort-on-prompts

    设置 :ref:`env.abort_on_prompts <abort-on-prompts>` 为 ``True``, 使Fabric在任何情况
    下提示输入时停止.

    .. versionadded:: 1.1

.. cmdoption:: -c RCFILE, --config=RCFILE

    设置 :ref:`env.rcfile <rcfile>` 为文件路径, Fabric在启动是试着加载并使用它更新环境变量.

.. cmdoption:: -d COMMAND, --display=COMMAND

    打印任务的入口文档，如果它存在. 目前不打印任务函数签名，所以描述文档是一个好办法.
    (They're *always* a good idea, of course -- just moreso here.)

.. cmdoption:: --connection-attempts=M, -n M

    设置尝试连接的次数. :ref:`env.connection_attempts <connection-attempts>`.

    .. seealso::
        :ref:`env.connection_attempts <connection-attempts>`,
        :ref:`env.timeout <timeout>`
    .. versionadded:: 1.4

.. cmdoption:: -D, --disable-known-hosts

    设置 :ref:`env.disable_known_hosts <disable-known-hosts>` 为 ``True``,
    阻止Fabric从用户的SSH :file:`known_hosts` 文件加载主机.

.. cmdoption:: -f FABFILE, --fabfile=FABFILE

    查找的匹配的的fabfile名称 (默认为 ``fabfile.py``), 或者一个明确的文件路径来作为fabfile加载
    (例如 ``/path/to/my/fabfile.py``.)

    .. seealso:: :doc:`fabfiles`

.. cmdoption:: -F LIST_FORMAT, --list-format=LIST_FORMAT

    可以控制输出格式 :option:`--list <-l>`. ``short`` 等同于 :option:`--shortlist`,
    ``normal`` 也是一样的只是忽略了该选项 (默认), 而 ``nested`` 打印出一个嵌套的命名空间树.

    .. versionadded:: 1.1
    .. seealso:: :option:`--shortlist`, :option:`--list <-l>`

.. cmdoption:: -g HOST, --gateway=HOST

    设置 :ref:`env.gateway <gateway>` 为 ``HOST`` 主机名.

    .. versionadded:: 1.5

.. cmdoption:: -h, --help

    输出标准的帮助信息, 所有可能的选项和它们所做的简要概述, 然后退出.

.. cmdoption:: --hide=LEVELS

    一个逗号分隔的列表 :doc:`output levels <output_controls>` 来隐藏to hide by
    default.


.. cmdoption:: -H HOSTS, --hosts=HOSTS

    设置 :ref:`env.hosts <hosts>` 为一个逗号分隔的主机名列表.

.. cmdoption:: -x HOSTS, --exclude-hosts=HOSTS

    设置 :ref:`env.exclude_hosts <exclude-hosts>` 指定逗号分隔的主机名列表用来排除
    主机.

    .. versionadded:: 1.1

.. cmdoption:: -i KEY_FILENAME

    当设置为文件路径，将加载文件作为SSH验证文件 (通常是一个私钥.) 这个选项能多次重复. 设置
    (或追加) :ref:`env.key_filename <key-filename>`.

.. cmdoption:: -I, --initial-password-prompt

    为了预先填写在会话开始前强制密码提示 (在fabfile加载喝选项解析之后，但是在执行任何任务前)
    :ref:`env.password <password>`.

    当通过 :option:`--password <-p>` 设置密码或在fabfile里设置 :ref:`env.password <password>`
    时...是有用的 (特别是并行会话, 在运行时输入是不可能的).

    .. note:: 通过在模块提供 :ref:`env.password <password>` 或 :option:`--password <-p>`提供的
        值会覆盖任何东西.

    .. seealso:: :ref:`password-management`

.. cmdoption:: -k

    设置 :ref:`env.no_keys <no_keys>` 为 ``True``, 强制SSH不在家目录寻找SSH私钥文件.

    .. versionadded:: 0.9.1

.. cmdoption:: --keepalive=KEEPALIVE

    设置 :ref:`env.keepalive <keepalive>` 为某个整数值, 特别是SSH存活时间间隔.

    .. versionadded:: 1.1

.. cmdoption:: --linewise

    强制输出缓存为行而不是字节. 通常有用或需要 :ref:`parallel execution <linewise-output>`.

    .. versionadded:: 1.3

.. cmdoption:: -l, --list

    正常导入fabfile, 然后打印所有找到的任务并退出. 也会答应每个任务描述的第一行, 如果存在，则继续
    (如果必要截断.)

    .. versionchanged:: 0.9.1
        增加文档字符串输出.
    .. seealso:: :option:`--shortlist`, :option:`--list-format <-F>`

.. cmdoption:: -p PASSWORD, --password=PASSWORD

    设置 :ref:`env.password <password>` 为一个字符串; 它将作为SSH连接或者调用
    ``sudo`` 程序的默认密码.

    .. seealso:: :option:`--initial-password-prompt <-I>`

.. cmdoption:: -P, --parallel

    设置 :ref:`env.parallel <env-parallel>` 为 ``True``, 将并行执行任务.

    .. versionadded:: 1.3
    .. seealso:: :doc:`/usage/parallel`

.. cmdoption:: --no-pty

    设置 :ref:`env.always_use_pty <always-use-pty>` 为 ``False``, 引起所有的
    `~fabric.operations.run`/`~fabric.operations.sudo` 的调用行为作为
    calls to behave as if
    one had specified ``pty=False``.

    .. versionadded:: 1.0

.. cmdoption:: -r, --reject-unknown-hosts

    设置 :ref:`env.reject_unknown_hosts <reject-unknown-hosts>` 为 ``True``,
    使Fabric在用户连接到SSH :file:`known_hosts` 文件中没有的主机时终止.

.. cmdoption:: -R ROLES, --roles=ROLES

    设置 :ref:`env.roles <roles>` 为一个逗号分隔的角色名列表.

.. cmdoption:: --set KEY=VALUE,...

    允许你设置默认的Fabric环境变量. 用这种方法设置的值有较低的优先级 -- 他们将不会覆盖
    更多通过命令行指定的环境变量. 例如::

        fab --set password=foo --password=bar

    的结果是 ``env.password = 'bar'``, 而不是 ``'foo'``

    多个 ``KEY=VALUE`` 通过逗号分隔, 例如. ``fab --set var1=val1,var2=val2``.

    比起基本字符变量, 你也可以设置环境变量为True通过略去 ``=VALUE`` (例如. ``fab --set KEY``),
    你也可以设置值为空字符串 (这样的到一个Faslse值) 通过保持等号，但略去 ``VALUE``
    (例如. ``fab --set KEY=``.)

    .. versionadded:: 1.4

.. cmdoption:: -s SHELL, --shell=SHELL

    设置 :ref:`env.shell <shell>` 为一个字符串, 覆盖默认的shell去执行远程命令.

.. cmdoption:: --shortlist

    类似于 :option:`--list <-l>`, 但没有任何的修饰, 只是新行分隔的任务名而没有锁进
    和文档字符.

    .. versionadded:: 0.9.2
    .. seealso:: :option:`--list <-l>`

.. cmdoption:: --show=LEVELS

    一个逗号分隔的列表 :doc:`output levels <output_controls>` 要添加的默认显示为.

    .. seealso:: `~fabric.operations.run`, `~fabric.operations.sudo`

.. cmdoption:: --ssh-config-path

    设置 :ref:`env.ssh_config_path <ssh-config-path>`.

    .. versionadded:: 1.4
    .. seealso:: :ref:`ssh-config`

.. cmdoption:: --skip-bad-hosts

    :ref:`env.skip_bad_hosts <skip-bad-hosts>`, 使Fabric跳过不可用的主机.

    .. versionadded:: 1.4

.. cmdoption:: --skip-unknown-tasks

    :ref:`env.skip_unknown_tasks <skip-unknown-tasks>`, 使Fabric跳过未知的任务.

    .. seealso::
        :ref:`env.skip_unknown_tasks <skip-unknown-tasks>`

.. cmdoption:: --timeout=N, -t N

    设置连接超时秒数. Sets :ref:`env.timeout <timeout>`.

    .. seealso::
        :ref:`env.timeout <timeout>`,
        :ref:`env.connection_attempts <connection-attempts>`
    .. versionadded:: 1.4

.. cmdoption:: --command-timeout=N, -T N

   设置远程命令执行超时秒数. Sets :ref:`env.command_timeout <command-timeout>`.

   .. seealso::
	:ref:`env.command_timeout <command-timeout>`,

   .. versionadded:: 1.6

.. cmdoption:: -u USER, --user=USER

    设置 :ref:`env.user <user>` 为一个字符串; 他将使用其作为SSH连接的默认用户名.

.. cmdoption:: -V, --version

    显示Fabric的版本号, 然后退出.

.. cmdoption:: -w, --warn-only

    设置 :ref:`env.warn_only <warn_only>` 为 ``True``, 使Fabric在遭遇命令的错误时
    继续执行.

.. cmdoption:: -z, --pool-size

    :ref:`env.pool_size <pool-size>`, 指定并发执行时有多少个进程同时执行.

    .. versionadded:: 1.3
    .. seealso:: :doc:`/usage/parallel`


.. _task-arguments:

单任务参数
=====

在 :ref:`command-line-options` 中的选项适用于 ``fab`` 作为一个整体调用;
即使顺序被打乱, 选项仍一样的给到所有任务. 此外, 由于任务都是Python函数,
它们通常在运行时才传递参数.

会带这些需要了解"单任务参数"的概念, 是一种特别的语法你可以附加任何到任务名的后面:

* 使用一个冒号 (``:``) 来分隔参数中的任务名;
* 使用逗号 (``,``) 来分隔另一个参数 (可以使用反斜线来转义, 例如. ``\,``);
* 使用等号 (``=``) 获取关键字参数, 或者忽略位置参数. 同样使用反斜线转义.

此外, 由于这个过程涉及字符串解析, 所有的值都以Python字符串的结果为准,
所以如此规划. (我们希望在未来版本的Fabric有所改进, 提供一个可以直观的
语法可以被找到.)

例如, 一个 "创建新用户" 的任务可以这样定义 (为了简便忽略了最实际的逻辑)::

    def new_user(username, admin='no', comment="No comment provided"):
        print("New User (%s): %s" % (username, comment))
        pass

你可以这样指定username::

    $ fab new_user:myusername

或者把它作为一个明确的关键字参数::

    $ fab new_user:username=myusername

如果同时给了两个参数, 你可以作为位置参数再次给予::

    $ fab new_user:myusername,yes

或者混合匹配, 就像使用Python::

    $ fab new_user:myusername,admin=yes

``print`` 调用在说明转义逗号时很有用, 比如::

    $ fab new_user:myusername,admin=no,comment='Gary\, new developer (starts Monday)'

.. note::
    给反斜线转义逗号加上引号是必需的, 因为不这样做会引起shell语法错误. 当一个参数
    包括shell相关的字符入空格时也需要引号.

上面所有都可以翻译为预期的Python函数调用. 例如. 最后的调用可以变为::

    >>> new_user('myusername', admin='yes', comment='Gary, new developer (starts Monday)')

角色和主机
-----

As mentioned in :ref:`the section on task execution <hosts-per-task-cli>`,
there are a handful of per-task keyword arguments (``host``, ``hosts``,
``role`` and ``roles``) which do not actually map to the task functions
themselves, but are used for setting per-task host and/or role lists.

These special kwargs are **removed** from the args/kwargs sent to the task
function itself; this is so that you don't run into TypeErrors if your task
doesn't define the kwargs in question. (It also means that if you **do** define
arguments with these names, you won't be able to specify them in this manner --
a regrettable but necessary sacrifice.)

.. note::

    If both the plural and singular forms of these kwargs are given, the value
    of the plural will win out and the singular will be discarded.

When using the plural form of these arguments, one must use semicolons (``;``)
since commas are already being used to separate arguments from one another.
Furthermore, since your shell is likely to consider semicolons a special
character, you'll want to quote the host list string to prevent shell
interpretation, e.g.::

    $ fab new_user:myusername,hosts="host1;host2"

Again, since the ``hosts`` kwarg is removed from the argument list sent to the
``new_user`` task function, the actual Python invocation would be
``new_user('myusername')``, and the function would be executed on a host list
of ``['host1', 'host2']``.

.. _fabricrc:

配置文件
====

Fabric currently honors a simple user settings file, or ``fabricrc`` (think
``bashrc`` but for ``fab``) which should contain one or more key-value pairs,
one per line. These lines will be subject to ``string.split('=')``, and thus
can currently only be used to specify string settings. Any such key-value pairs
will be used to update :doc:`env <env>` when ``fab`` runs, and is loaded prior
to the loading of any fabfile.

By default, Fabric looks for ``~/.fabricrc``, and this may be overridden by
specifying the :option:`-c` flag to ``fab``.

For example, if your typical SSH login username differs from your workstation
username, and you don't want to modify ``env.user`` in a project's fabfile
(possibly because you expect others to use it as well) you could write a
``fabricrc`` file like so::

    user = ssh_user_name

Then, when running ``fab``, your fabfile would load up with ``env.user`` set to
``'ssh_user_name'``. Other users of that fabfile could do the same, allowing
the fabfile itself to be cleanly agnostic regarding the default username.
