=============
Fabfile的构造和使用
=============

本文档关于fabfile的各部分讲解，包括如何最好的编写，怎样使用.

.. _fabfile-discovery:

Fabfile 发现
==========

Fabric 可以加载Python模块 (例如. ``fabfile.py``) 或者包 (例如. 一个包含 ``__init__.py`` 文件的
``fabfile/`` 目录).
directory containing an ``__init__.py``). 默认情况它寻找名为 ``fabfile`` 的文件
- 不论 ``fabfile/`` 还是 ``fabfile.py``.

fabfile 搜索算法是寻找在调用用户当前的工作目录或任何父目录. 因此是面向 "工程"
来使用，例如将 ``fabfile.py`` 放置到源码的根目录. 这样一个fabfile会被发现不论用户在哪里
调用 ``fab``.

指定的名称可以用命令行参数 :option:`-f` 重写, 或者通过在fabfile添加 :ref:`fabricrc <fabricrc>`
行. 例如, 如果你想要命名fabfile为 ``fab_tasks.py``, 可以创建这样一个文件然后执行
``fab -f fab_tasks.py <task name>``, 或添加 ``fabfile = fab_tasks.py`` 到
``~/.fabricrc``.

如果给予的fabfile名称包含路径而不只是一个文件名 (如. ``../fabfile.py`` 或
``/dir1/dir2/custom_fabfile``) 将会当作文件路径并检查是否存在而不进行任何查找.
在这种模式下波浪线将被使用, 参考另一个例子如 ``~/personal_fabfile.py``.

.. note::

    Fabric 为了访问你的fabfile使用一般的 ``import`` (实际上是 ``__import__``)
    -- 它没有执行任何 ``eval``-ing 或类似的操作. 为了正常工作，Fabric临时添加
    发现包含的fabfile的文件夹到Python加载路径 (随后立即删除.)

.. versionchanged:: 0.9.2
    可以加载包fabfile.


.. _importing-the-api:

在Fabric中导入
==========

因为Fabric就是Python，你 *可以* 导入需要的任何组件. 然而，为了封装和方便的目的
(为了使包装Fabric的脚本的生活更轻松) Fabric有公共API包含在 ``fabric.api`` 模块.

所有Fabric的 :doc:`../api/core/operations`, :doc:`../api/core/context_managers`,
:doc:`../api/core/decorators` 和 :doc:`../api/core/utils` 都被独立的包含在这个模块
独立的命名空间. 在你的fabfile里非常简单的开启和一致的接口::

    from fabric.api import *

    # call run(), sudo(), etc etc

这并不是技术上的最佳事件 (for `a number of reasons`_)
如果你只是使用几个Fab API 调用, 大概 *是* 一个好的主意明确的导入 ``from fabric.api import env, run``
或者类似的. 然后在通常的fabfiles 可能会使用全部或大多数API并导入::

    from fabric.api import *

这样将会更易读写比起::

    from fabric.api import abort, cd, env, get, hide, hosts, local, prompt, \
        put, require, roles, run, runs_once, settings, show, sudo, warn

所以在这种情况下我们认为实用比最佳时间更重要.

.. _a number of reasons: http://python.net/~goodger/projects/pycon/2007/idiomatic/handout.html#importing


定义任务并导入调用
=========

关于Fabric在加载fabfile时作为任务导入的更多信息,
以及如何更好的导入其他代码请参阅 :doc:`/usage/tasks` 在 :doc:`execution` 文档.
