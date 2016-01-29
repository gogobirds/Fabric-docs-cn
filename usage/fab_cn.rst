=============
``fab`` 选项和参数
=============

通过命令行工具 ``fab`` 最通用的利用Fabric的方式, 当Fabric安装后将会存在于你的
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

正如在 :ref:`the section on task execution <hosts-per-task-cli>` 中提到的,
有少数的单任务参数 (``host``, ``hosts``, ``role`` and ``roles``) 实际上不能映射到任务函数本身,
但是可以用来设置每个任务的主机或角色列表.

可以发送到函数的参数中删除特定参数; 如此可以让你不运行出TypeError如果你的任务没有在问题中声明参数.
(It also means that if you **do** define arguments with these names, you won't be able to specify them in this manner --
a regrettable but necessary sacrifice.)

.. note::

    如果这些参数的单数和复数形式被同时给出, 该值的复数形式会胜出并且单数形式会被丢弃.

当使用这些参数的复数形式, 必须使用分号 (``;``) 因为逗号已经被用来彼此分离.
此外, 由于你的shell可能使用分号为一个特殊字符, 你可能需要引号将主机列表字符串引用以防止shell解释,
例如::

    $ fab new_user:myusername,hosts="host1;host2"

这样, 只要``hosts`` 参数从参数列表中移除并发送给 ``new_user`` 任务函数, 实际的Python调用是这样的
``new_user('myusername')``, 这个函数将会在主机列表 ``['host1', 'host2']`` 上执行.

.. _fabricrc:

配置文件
====

Fabric 目前有一个简单的用户设置文件,``fabricrc`` (类似 ``bashrc`` 但是为 ``fab`` 服务)
能够每行包括一个或多个键值对. 行受限于 ``string.split('=')``, 因此目前只能用于指定字符串设置.
任何这样的键值对将在 ``fab`` 更新 :doc:`env <env>`, 并在任何fabfile前加载.

默认情况下, Fabric搜索 ``~/.fabricrc``, 可以通过选项 :option:`-c` 指定给 ``fab``.

例如, 如果你用SSH登录用户名与工作台用户名不同, 你也可能不想在以工程的fabfile里修改
``env.user``, (或者因为你想使用其他更好的方式) 你可以像这样写入到 ``fabricrc`` 文件::

    user = ssh_user_name

接着, 当运行 ``fab``, fabfile将加载 ``env.user`` 为 ``'ssh_user_name'``.
其他用户使用fabfile也可以这样做, 让fabfile对于默认用户名是干净不可知的.
