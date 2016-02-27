=====
SSH行为
=====

Fabric目前使用纯Python的SSH重实现来管理连接, 这意味着偶尔会由于库的功能而受到限制.
以下是值得注意的地方, Fabric会展现不符的行为, 或灵活如``ssh``命令行程序的行为.


未知主机
====

SSH主机秘钥的跟踪机制会密切关注所有你尝试连接到的主机, 并通过标识符之间的映射维护 ``~/.ssh/known_hosts`` 文件
(IP地址, 有时也有主机名) 和SSH秘钥. (更多细节可参见 `OpenSSH documentation
<http://openssh.org/manual.html>`_.)

``paramiko`` 库能够加载你的 ``known_hosts`` 文件, 然后通过映射比较任何连接的主机.
设置可用于确定当前的未知主机发生了什么 (在 ``known_hosts``中没有找到用户名和IP的主机):

* **Reject**: 主机秘钥被拒绝且无法连接. 这导致一个Python异常, 这会附带着未知主机的消息终止你的Fabric会话.
* **Add**: 新主机秘钥添加到内存中的已知主机列表, 连接正常, 一切继续. 注意这并**没有**修改磁盘上的``known_hosts`` 文件!
* **Ask**: 没有在Fabric级别实现, 这是一个 ``paramiko`` 库选项, 这将导致用户被提示未知秘钥和是否接受.

如上, 是否拒绝或添加主机as above, 在Fabric里通过 :ref:`env.reject_unknown_hosts <reject-unknown-hosts>` 选项控制,
为了方便默认为False. 这是便捷和安全之间的有效平衡; 所有感受到这种平衡的人都可以轻松地在模块级别
设置``env.reject_unknown_hosts = True``以修改他们的fabfiles.


改变了密码的已知主机
==========

SSH的键/指纹追踪可以检测到中间人攻击: 如果攻击者重定向你的SSH连接到他控制的电脑上, 并且伪装成你原来的目的服务器,
不会匹配主机秘钥. 因此, 当 ``known_hosts`` 上以前记录的主机发送了一个不同的主机秘钥时,
SSH (和它的Python实现)的默认行为是立即中断连接.

在一些边缘情况如EC2部署中, 你可能会想要忽略这个潜在问题. 在写这篇文章的时候, 我们的SSH层不允许我们控制当前行为,
但我们可以简单地通过 ``known_hosts`` 的加载来回避它 -- 如果比较的主机列表为空, 就没有问题.
想要这样的时候就将 :ref:`env.disable_known_hosts <disable-known-hosts>` 设为True; 默认为False,以保持默认的SSH行为.

.. warning::
    启用 :ref:`env.disable_known_hosts <disable-known-hosts>` 会让你敞陆于中间人攻击之下! 请小心使用.
