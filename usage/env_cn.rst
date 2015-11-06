================
�����ֵ�, ``env``
================
Fabricһ����ȴ�������ĸ����Ϊ�ǡ������� :
    һ��python���ֵ�����,���������������ע�����ڲ�����Ĺ������ݿռ�.

�����ֵ䱻��Ϊ��ȫ�ֱ�����``fabric.state.env`` Ϊʹ�÷���Ҳ��Ƕ�� ``fabric.api``.
``env`` �ؼ�����ʱҲ����Ϊ "env variables".

��������
=======
Fabric�Ĵ󲿷ֲ�������ͨ���޸� ``env`` ����������, ���� ``env.hosts`` (�μ� :ref:`the tutorial <defining-connections>`).
�����������޸ĵı������£�

* ``user``: ����SSH����ʱ��FabricĬ��ʹ�ñ����û���,����б�Ҫ����ʹ�� ``env.user`` ���Ǵ˱���.
    ���������ÿ�������������û������μ� :doc:`execution`.
* ``password``: ������ʾ����Ĭ�����ӻ���sudo���롣�����˱���δ���û���Ч��Fabric��������.
* ``warn_only``: ��Զ���ն˼�⵽����ʱ������Fabric�Ƿ��˳��Ĳ�������. ������Ϣ�μ� :doc:`execution`.

�����������������������б��μ����ĵ�ĩβ.

`~fabric.context_managers.settings` �����Ĺ�����
---------------------------------------------

�ܶ�ʱ��Ϊ���ڸ��������ñ䶯ֻ���һ������飬��ʱ�޸� ``env`` ��ú�����.
Fabric�ṩ`~fabric.context_managers.settings`�����Ĺ�����������������"��/ֵ"�ԣ�
�����������޸ķ�װ�õĴ�����е� ``env``.

���磬�ܶ����������� ``warn_only`` �������ã����£�.
�����ṩ���������д��룬ʹ�� ``settings(warn_only=True)``��
�μ��򻯰�� ``contrib`` `~fabric.contrib.files.exists` ����::

    from fabric.api import settings, run

    def exists(path):
        with settings(warn_only=True):
            return run('test -e %s' % path)

���������Ĺ��������������ƵĹ��ߣ���μ� :doc:`../api/core/context_managers` API�ĵ�.

��������״̬
===========

�Ѿ��ᵽ����``env`` ���󴿴��Ǹ��ֵ����࣬�������Լ���fabfile����Ҳ���Դ洢��Ϣ.
��ʱ�ڶ������䱣��״̬��ֻ����һ����������ͱ�ú�������.

.. note::����ע����

    ``env``�����������ʷ�Եģ�����ǰ��fabfiles���Ǵ�python��д����˻�����������佻����Ϣ��Ψһ·��.
    ��������ֱ�ӵ�����������������̣���������Ҫʱ����ģ�鼶����״̬.

    ���Ժ�İ汾�Fabric���̻߳����ȫ��
    ������һ�㣬 ``env``���ܻ��Ϊ����ȫ��״̬��Ψһ��㡢��ȫ;��.

��������
=======

�ڼ̳�``dict``ʱ��Fabric�� ``env`` Ҳ�����޸ģ��Ա�������ֵ����ͨ�����Է�������д��������������.
���仰˵��``env.host_string``��``env['host_string']``��������ͬ.
ͨ�����ǻᷢ�֣����Է��ʵķ�ʽ�ܼ����ü��̵Ĺ�����������ߴ���Ŀɶ���.
�����Ҳ�Ǻ�``env``�������Ƽ���ʽ.

ʵ�����ֵ���������;��Ҳ�����ã�����Python���� ``dict``���ַ��������������Ҫ��һ���ַ����в�������������ʱ�ͺܱ����.
ʹ��"��ͨ"�ַ�������Ϳ�������������::

    print("Executing on %s as %s" % (env.host, env.user))

ʹ���ֵ��������͸��ɶ������::

    print("Executing on %(host)s as %(user)s" % env)

.. _env-vars:

���������������б�
================

����������Ԥ���壨����Fabric����ʱ�Զ��壩�Ļ�������.
�����д������ֱ�Ӳ���ʱ��һ�����ʹ�� ~fabric.context_managers`,
����`~fabric.context_managers.settings`�����ض��������Ĺ�����������`~fabric.context_managers.cd`.

��Ҫע����������еĺܶ඼����ͨ��``fab``�������в��������ã��μ�:doc:`fab`.
�ʵ��ĵط��ṩ�н������á�Cross-references��.

.. seealso:: :option:`--set`

.. _abort-exception:

``abort_exception``
-------------------

**Default:** ``None``

Fabric normally handles aborting by printing an error message to stderr and
calling ``sys.exit(1)``. This setting allows you to override that behavior
(which is what happens when ``env.abort_exception`` is ``None``.)

Give it a callable which takes a string (the error message that would have been
printed) and returns an exception instance.  That exception object is then
raised instead of ``SystemExit`` (which is what ``sys.exit`` does.)

Much of the time you'll want to simply set this to an exception class, as those
fit the above description perfectly (callable, take a string, return an
exception instance.) E.g. ``env.abort_exception = MyExceptionClass``.

.. _abort-on-prompts:

``abort_on_prompts``
--------------------

**Default:** ``False``

When ``True``, Fabric will run in a non-interactive mode, calling
`~fabric.utils.abort` anytime it would normally prompt the user for input (such
as password prompts, "What host to connect to?" prompts, fabfile invocation of
`~fabric.operations.prompt`, and so forth.) This allows users to ensure a Fabric
session will always terminate cleanly instead of blocking on user input forever
when unforeseen circumstances arise.

.. versionadded:: 1.1
.. seealso:: :option:`--abort-on-prompts`


``all_hosts``
-------------

**Default:** ``[]``

Set by ``fab`` to the full host list for the currently executing command. For
informational purposes only.

.. seealso:: :doc:`execution`

.. _always-use-pty:

``always_use_pty``
------------------

**Default:** ``True``

When set to ``False``, causes `~fabric.operations.run`/`~fabric.operations.sudo`
to act as if they have been called with ``pty=False``.

.. seealso:: :option:`--no-pty`
.. versionadded:: 1.0

.. _colorize-errors:

``colorize_errors``
-------------------

**Default** ``False``

When set to ``True``, error output to the terminal is colored red and warnings
are colored magenta to make them easier to see.

.. versionadded:: 1.7

.. _combine-stderr:

``combine_stderr``
------------------

**Default**: ``True``

Causes the SSH layer to merge a remote program's stdout and stderr streams to
avoid becoming meshed together when printed. See :ref:`combine_streams` for
details on why this is needed and what its effects are.

.. versionadded:: 1.0

``command``
-----------

**Default:** ``None``

Set by ``fab`` to the currently executing command name (e.g., when executed as
``$ fab task1 task2``, ``env.command`` will be set to ``"task1"`` while
``task1`` is executing, and then to ``"task2"``.) For informational purposes
only.

.. seealso:: :doc:`execution`

``command_prefixes``
--------------------

**Default:** ``[]``

Modified by `~fabric.context_managers.prefix`, and prepended to commands
executed by `~fabric.operations.run`/`~fabric.operations.sudo`.

.. versionadded:: 1.0

.. _command-timeout:

``command_timeout``
-------------------

**Default:** ``None``

Remote command timeout, in seconds.

.. versionadded:: 1.6
.. seealso:: :option:`--command-timeout`

.. _connection-attempts:

``connection_attempts``
-----------------------

**Default:** ``1``

Number of times Fabric will attempt to connect when connecting to a new server. For backwards compatibility reasons, it defaults to only one connection attempt.

.. versionadded:: 1.4
.. seealso:: :option:`--connection-attempts`, :ref:`timeout`

``cwd``
-------

**Default:** ``''``

Current working directory. Used to keep state for the
`~fabric.context_managers.cd` context manager.

.. _dedupe_hosts:

``dedupe_hosts``
----------------

**Default:** ``True``

Deduplicate merged host lists so any given host string is only represented once
(e.g. when using combinations of ``@hosts`` + ``@roles``, or ``-H`` and
``-R``.)

When set to ``False``, this option relaxes the deduplication, allowing users
who explicitly want to run a task multiple times on the same host (say, in
parallel, though it works fine serially too) to do so.

.. versionadded:: 1.5

.. _disable-known-hosts:

``disable_known_hosts``
-----------------------

**Default:** ``False``

If ``True``, the SSH layer will skip loading the user's known-hosts file.
Useful for avoiding exceptions in situations where a "known host" changing its
host key is actually valid (e.g. cloud servers such as EC2.)

.. seealso:: :option:`--disable-known-hosts <-D>`, :doc:`ssh`


.. _eagerly-disconnect:

``eagerly_disconnect``
----------------------

**Default:** ``False``

If ``True``, causes ``fab`` to close connections after each individual task
execution, instead of at the end of the run. This helps prevent a lot of
typically-unused network sessions from piling up and causing problems with
limits on per-process open files, or network hardware.

.. note::
    When active, this setting will result in the disconnect messages appearing
    throughout your output, instead of at the end. This may be improved in
    future releases.

.. _effective_roles:

``effective_roles``
-------------------

**Default:** ``[]``

Set by ``fab`` to the roles list of the currently executing command. For
informational purposes only.

.. versionadded:: 1.9
.. seealso:: :doc:`execution`

.. _exclude-hosts:

``exclude_hosts``
-----------------

**Default:** ``[]``

Specifies a list of host strings to be :ref:`skipped over <exclude-hosts>`
during ``fab`` execution. Typically set via :option:`--exclude-hosts/-x <-x>`.

.. versionadded:: 1.1


``fabfile``
-----------

**Default:** ``fabfile.py``

Filename pattern which ``fab`` searches for when loading fabfiles.
To indicate a specific file, use the full path to the file. Obviously, it
doesn't make sense to set this in a fabfile, but it may be specified in a
``.fabricrc`` file or on the command line.

.. seealso:: :option:`--fabfile <-f>`, :doc:`fab`


.. _gateway:

``gateway``
-----------

**Default:** ``None``

Enables SSH-driven gatewaying through the indicated host. The value should be a
normal Fabric host string as used in e.g. :ref:`env.host_string <host_string>`.
When this is set, newly created connections will be set to route their SSH
traffic through the remote SSH daemon to the final destination.

.. versionadded:: 1.5

.. seealso:: :option:`--gateway <-g>`


.. _host_string:

``host_string``
---------------

**Default:** ``None``

Defines the current user/host/port which Fabric will connect to when executing
`~fabric.operations.run`, `~fabric.operations.put` and so forth. This is set by
``fab`` when iterating over a previously set host list, and may also be
manually set when using Fabric as a library.

.. seealso:: :doc:`execution`


.. _forward-agent:

``forward_agent``
--------------------

**Default:** ``False``

If ``True``, enables forwarding of your local SSH agent to the remote end.

.. versionadded:: 1.4

.. seealso:: :option:`--forward-agent <-A>`

.. _host:

``host``
--------

**Default:** ``None``

Set to the hostname part of ``env.host_string`` by ``fab``. For informational
purposes only.

.. _hosts:

``hosts``
---------

**Default:** ``[]``

The global host list used when composing per-task host lists.

.. seealso:: :option:`--hosts <-H>`, :doc:`execution`

.. _keepalive:

``keepalive``
-------------

**Default:** ``0`` (i.e. no keepalive)

An integer specifying an SSH keepalive interval to use; basically maps to the
SSH config option ``ServerAliveInterval``. Useful if you find connections are
timing out due to meddlesome network hardware or what have you.

.. seealso:: :option:`--keepalive`
.. versionadded:: 1.1


.. _key:

``key``
----------------

**Default:** ``None``

A string, or file-like object, containing an SSH key; used during connection
authentication.

.. note::
    The most common method for using SSH keys is to set :ref:`key-filename`.

.. versionadded:: 1.7


.. _key-filename:

``key_filename``
----------------

**Default:** ``None``

May be a string or list of strings, referencing file paths to SSH key files to
try when connecting. Passed through directly to the SSH layer. May be
set/appended to with :option:`-i`.

.. seealso:: `Paramiko's documentation for SSHClient.connect() <http://docs.paramiko.org/en/latest/api/client.html#paramiko.client.SSHClient.connect>`_

.. _env-linewise:

``linewise``
------------

**Default:** ``False``

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

**Default:** ``False``

If ``True``, will tell the SSH layer not to seek out running SSH agents when
using key-based authentication.

.. versionadded:: 0.9.1
.. seealso:: :option:`--no_agent <-a>`

.. _no_keys:

``no_keys``
------------------

**Default:** ``False``

If ``True``, will tell the SSH layer not to load any private key files from
one's ``$HOME/.ssh/`` folder. (Key files explicitly loaded via ``fab -i`` will
still be used, of course.)

.. versionadded:: 0.9.1
.. seealso:: :option:`-k`

.. _env-parallel:

``parallel``
-------------------

**Default:** ``False``

When ``True``, forces all tasks to run in parallel. Implies :ref:`env.linewise
<env-linewise>`.

.. versionadded:: 1.3
.. seealso:: :option:`--parallel <-P>`, :doc:`parallel`

.. _password:

``password``
------------

**Default:** ``None``

The default password used by the SSH layer when connecting to remote hosts,
**and/or** when answering `~fabric.operations.sudo` prompts.

.. seealso:: :option:`--initial-password-prompt <-I>`, :ref:`env.passwords <passwords>`, :ref:`password-management`

.. _passwords:

``passwords``
-------------

**Default:** ``{}``

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

**Default:** ``''``

Used to set the ``$PATH`` shell environment variable when executing commands in
`~fabric.operations.run`/`~fabric.operations.sudo`/`~fabric.operations.local`.
It is recommended to use the `~fabric.context_managers.path` context manager
for managing this value instead of setting it directly.

.. versionadded:: 1.0


.. _pool-size:

``pool_size``
-------------

**Default:** ``0``

Sets the number of concurrent processes to use when executing tasks in parallel.

.. versionadded:: 1.3
.. seealso:: :option:`--pool-size <-z>`, :doc:`parallel`

.. _prompts:

``prompts``
-------------

**Default:** ``{}``

The ``prompts`` dictionary allows users to control interactive prompts. If a
key in the dictionary is found in a command's standard output stream, Fabric
will automatically answer with the corresponding dictionary value.

.. versionadded:: 1.9

.. _port:

``port``
--------

**Default:** ``None``

Set to the port part of ``env.host_string`` by ``fab`` when iterating over a
host list. May also be used to specify a default port.

.. _real-fabfile:

``real_fabfile``
----------------

**Default:** ``None``

Set by ``fab`` with the path to the fabfile it has loaded up, if it got that
far. For informational purposes only.

.. seealso:: :doc:`fab`


.. _remote-interrupt:

``remote_interrupt``
--------------------

**Default:** ``None``

Controls whether Ctrl-C triggers an interrupt remotely or is captured locally,
as follows:

* ``None`` (the default): only `~fabric.operations.open_shell` will exhibit
  remote interrupt behavior, and
  `~fabric.operations.run`/`~fabric.operations.sudo` will capture interrupts
  locally.
* ``False``: even `~fabric.operations.open_shell` captures locally.
* ``True``: all functions will send the interrupt to the remote end.

.. versionadded:: 1.6


.. _rcfile:

``rcfile``
----------

**Default:** ``$HOME/.fabricrc``

Path used when loading Fabric's local settings file.

.. seealso:: :option:`--config <-c>`, :doc:`fab`

.. _reject-unknown-hosts:

``reject_unknown_hosts``
------------------------

**Default:** ``False``

If ``True``, the SSH layer will raise an exception when connecting to hosts not
listed in the user's known-hosts file.

.. seealso:: :option:`--reject-unknown-hosts <-r>`, :doc:`ssh`

.. _system-known-hosts:

``system_known_hosts``
------------------------

**Default:** ``None``

If set, should be the path to a :file:`known_hosts` file.  The SSH layer will
read this file before reading the user's known-hosts file.

.. seealso:: :doc:`ssh`

.. _roledefs:

``roledefs``
------------

**Default:** ``{}``

Dictionary defining role name to host list mappings.

.. seealso:: :doc:`execution`

.. _roles:

``roles``
---------

**Default:** ``[]``

The global role list used when composing per-task host lists.

.. seealso:: :option:`--roles <-R>`, :doc:`execution`

.. _shell:

``shell``
---------

**Default:** ``/bin/bash -l -c``

Value used as shell wrapper when executing commands with e.g.
`~fabric.operations.run`. Must be able to exist in the form ``<env.shell>
"<command goes here>"`` -- e.g. the default uses Bash's ``-c`` option which
takes a command string as its value.

.. seealso:: :option:`--shell <-s>`,
             :ref:`FAQ on bash as default shell <faq-bash>`, :doc:`execution`

.. _skip-bad-hosts:

``skip_bad_hosts``
------------------

**Default:** ``False``

If ``True``, causes ``fab`` (or non-``fab`` use of `~fabric.tasks.execute`) to skip over hosts it can't connect to.

.. versionadded:: 1.4
.. seealso::
    :option:`--skip-bad-hosts`, :ref:`excluding-hosts`, :doc:`execution`


.. _skip-unknown-tasks:

``skip_unknown_tasks``
----------------------

**Default:** ``False``

If ``True``, causes ``fab`` (or non-``fab`` use of `~fabric.tasks.execute`)
to skip over tasks not found, without aborting.

.. seealso::
    :option:`--skip-unknown-tasks`


.. _ssh-config-path:

``ssh_config_path``
-------------------

**Default:** ``$HOME/.ssh/config``

Allows specification of an alternate SSH configuration file path.

.. versionadded:: 1.4
.. seealso:: :option:`--ssh-config-path`, :ref:`ssh-config`

``ok_ret_codes``
------------------------

**Default:** ``[0]``

Return codes in this list are used to determine whether calls to
`~fabric.operations.run`/`~fabric.operations.sudo`/`~fabric.operations.sudo`
are considered successful.

.. versionadded:: 1.6

.. _sudo_prefix:

``sudo_prefix``
---------------

**Default:** ``"sudo -S -p '%(sudo_prompt)s' " % env``

The actual ``sudo`` command prefixed onto `~fabric.operations.sudo` calls'
command strings. Users who do not have ``sudo`` on their default remote
``$PATH``, or who need to make other changes (such as removing the ``-p`` when
passwordless sudo is in effect) may find changing this useful.

.. seealso::

    The `~fabric.operations.sudo` operation; :ref:`env.sudo_prompt
    <sudo_prompt>`

.. _sudo_prompt:

``sudo_prompt``
---------------

**Default:** ``"sudo password:"``

Passed to the ``sudo`` program on remote systems so that Fabric may correctly
identify its password prompt.

.. seealso::

    The `~fabric.operations.sudo` operation; :ref:`env.sudo_prefix
    <sudo_prefix>`

.. _sudo_user:

``sudo_user``
-------------

**Default:** ``None``

Used as a fallback value for `~fabric.operations.sudo`'s ``user`` argument if
none is given. Useful in combination with `~fabric.context_managers.settings`.

.. seealso:: `~fabric.operations.sudo`

.. _env-tasks:

``tasks``
-------------

**Default:** ``[]``

Set by ``fab`` to the full tasks list to be executed for the currently
executing command. For informational purposes only.

.. seealso:: :doc:`execution`

.. _timeout:

``timeout``
-----------

**Default:** ``10``

Network connection timeout, in seconds.

.. versionadded:: 1.4
.. seealso:: :option:`--timeout`, :ref:`connection-attempts`

``use_shell``
-------------

**Default:** ``True``

Global setting which acts like the ``shell`` argument to
`~fabric.operations.run`/`~fabric.operations.sudo`: if it is set to ``False``,
operations will not wrap executed commands in ``env.shell``.


.. _use-ssh-config:

``use_ssh_config``
------------------

**Default:** ``False``

Set to ``True`` to cause Fabric to load your local SSH config file.

.. versionadded:: 1.4
.. seealso:: :ref:`ssh-config`


.. _user:

``user``
--------

**Default:** User's local username

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

**Default:** current Fabric version string

Mostly for informational purposes. Modification is not recommended, but
probably won't break anything either.

.. seealso:: :option:`--version <-V>`

.. _warn_only:

``warn_only``
-------------

**Default:** ``False``

Specifies whether or not to warn, instead of abort, when
`~fabric.operations.run`/`~fabric.operations.sudo`/`~fabric.operations.local`
encounter error conditions.

.. seealso:: :option:`--warn-only <-w>`, :doc:`execution`
