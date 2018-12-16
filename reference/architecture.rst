架构
============

本章概述了Doctrine2的整体架构，术语和约束。建议仔细阅读本章。

使用对象-关系映射器
---------------------------------

正如ORM已经暗示的那样，Doctrine2旨在简化数据库行和PHP对象模型之间的转换。
因此，Doctrine的主要用例是使用面向对象编程范例的应用。对于主要不使用对象的应用，Doctrine2不太适合。

要求
------------

Doctrine2要求最低PHP 5.4。为了大幅度提高性能，还建议你使用APC和PHP。

Doctrine2软件包
-------------------

Doctrine2分为三个主要软件包。

-  Common
-  DBAL (包含 Common)
-  ORM (包含 DBAL+Common)

本手册主要介绍ORM包，有时涉及底层DBAL和Common包的部分内容。
由于一些原因，Doctrine代码库被分成这些包，它们是......

-  ...使事情更易于维护和分离
-  ...DBAL允许你在不用ORM或DBAL的情况下使用Doctrine Common中的代码
-  ...允许你在不用ORM的情况下使用DBAL

Common软件包
~~~~~~~~~~~~~~~~~~

Common软件包含高度可重用的组件，这些组件除了软件包本身（当然还有PHP）之外没有其他依赖关系。
Common软件包的根命名空间是 ``Doctrine\Common``。

BAL软件包
~~~~~~~~~~~~~~~~

DBAL软件包在PDO之上包含增强的数据库抽象层，但不强烈(strongly)绑定到PDO。
此层的目的是提供一个API，它可以弥合不同RDBMS供应商之间的大部分差异。DBAL软件包的根命名空间是
``Doctrine\DBAL``。

ORM软件包
~~~~~~~~~~~~~~~

ORM软件包中包含对象-关系映射工具包，它为普通PHP对象提供透明的关系持久性。
ORM软件包的根命名空间是 ``Doctrine\ORM``。

术语
-----------

实体
~~~~~~~~

实体是一个轻量级的、持久的域对象。一个实体可以是遵循以下限制的任何常规PHP类：

-  实体类不能是 ``final`` 或包含 ``final`` 方法。
-  任何实体类的所有持久属性/字段应始终为 ``private`` 或 ``protected``，否则延迟加载可能无法按预期工作。
   因为你要序列化实体（例如会话）时，它的属性应该是 ``protected`` （请参阅下面的序列化部分）。
-  实体类不能实现 ``__clone``，但可以 :doc:`安全的实现它 <../cookbook/implementing-wakeup-or-clone>`。
-  实体类不能实现 ``__wakeup``，但可以 :doc:`安全的实现它 <../cookbook/implementing-wakeup-or-clone>`。
   另外也可以以实现
   `Serializable <http://php.net/manual/en/class.serializable.php>`_ 来替代。
-  直接或间接继承类层级中的任何两个实体类都不能具有相同名称的映射属性。
   也就是说，如果 ``B`` 继承了 ``A``，则 ``B`` 不能具有与从 ``A`` 继承的已映射字段相同名称的映射字段。
   Any two entity classes in a class hierarchy that inherit
   directly or indirectly from one another must not have a mapped
   property with the same name.
   That is, if B inherits from A then B
   must not have a mapped field with the same name as an already
   mapped field that is inherited from A.
-  实体不能使用 ``func_get_args()`` 来实现变量参数。
   出于性能原因，生成的代理不支持此操作，并且在违反此限制时，你的代码实际上可能无法正常工作。

实体支持继承、多态关联和多态查询。抽象类和具体类都可以是实体。
实体可以继承非实体类以及实体类，非实体类可以继承实体类。

.. note::

    只有在 *你* 使用 *new* 关键字构造新实例时才会调用该实体的构造函数。
    Doctrine从不调用实体的构造函数，因此你可以根据需要自由使用它们，甚至可以使用任何类型的参数。

实体状态
~~~~~~~~~~~~~

一个实体实例可以被表示为 ``NEW``、``MANAGED``、``DETACHED`` 或 ``REMOVED``。

-  ``NEW`` 实体实例没有持久标识，并且尚未与 ``EntityManager`` 和 ``UnitOfWork``
   关联（即刚刚使用 ``new`` 运算符创建的实体）。
-  ``MANAGED`` 实体实例是一个具有与 ``EntityManager`` 关联的持久标识的实例，因此其持久性已被管理。
-  ``DETACHED`` 实体实例是一个具有不（或不再）与 ``EntityManager`` 和 ``UnitOfWork`` 相关联的持久标识的实例。
-  ``REMOVED`` 实体实例是一个具有与 ``EntityManager`` 关联的持久标识的实例，该实例将在事务提交时从数据库中删除。

.. _architecture_persistent_fields:

持久字段
~~~~~~~~~~~~~~~~~

实体的持久状态由实例变量表示。实例变量必须仅由实体实例本身在实体的方法内直接访问。
实体的客户端不得访问实例变量。客户端只能通过实体的方法（即访问器方法：getter/setter）或其他业务方法来获取实体的状态。

必须根据 ``Doctrine\Common\Collections\Collection`` 接口来定义集合值(Collection-valued)的持久字段和属性。
应用可以使用集合实现的类型在实体持久化之前初始化字段或属性。一旦实体被管理（或分离），后续访问必须通过该接口类型。

序列化实体
~~~~~~~~~~~~~~~~~~~~

序列化实体可能会有问题，并且不是真正推荐的，至少只要实体实例仍然保留对代理对象的引用或仍然由 ``EntityManager`` 管理。
如果你打算序列化（和反序列化）仍然保存对代理对象的引用的实体实例，则由于技术限制，你可能会遇到私有属性的问题。
代理对象实现了 ``__sleep`` 并且不可能在父类中为 ``__sleep`` 返回私有属性的名称。
另一方面，它不是代理对象实现 ``Serializable`` 的解决方案，因为不能很好地处理任何潜在的循环对象引用（至少我们没有找到方法，如果你这样做，请联系我们）。
Serializing entities can be problematic and is not really
recommended, at least not as long as an entity instance still holds
references to proxy objects or is still managed by an
EntityManager. If you intend to serialize (and unserialize) entity
instances that still hold references to proxy objects you may run
into problems with private properties because of technical
limitations. Proxy objects implement ``__sleep`` and it is not
possible for ``__sleep`` to return names of private properties in
parent classes. On the other hand it is not a solution for proxy
objects to implement ``Serializable`` because Serializable does not
work well with any potential cyclic object references (at least we
did not find a way yet, if you did, please contact us).

EntityManager
~~~~~~~~~~~~~~~~~

``EntityManager`` 类是Doctrine2提供的ORM功能的中心访问点。
``EntityManager`` API用于管理对象的持久性和查询持久对象。

事务性后写
~~~~~~~~~~~~~~~~~~~~~~~~~~

一个 ``EntityManager`` 和底层 ``UnitOfWork`` 采用一种称为“事务性后写”的策略。该策略延迟SQL语句的执行，以便以最有效的方式执行它们；然后在事务结束时执行它们，以便快速的释放所有写锁。
你应该将Doctrine视为一种工具，用于在明确定义的工作单元中将内存中的对象与数据库同步。
处理你的对象并像往常一样修改它们，并在完成调用 ``EntityManager#flush()`` 以使你的更改持久化。

工作单元
~~~~~~~~~~~~~~~~

一个 ``EntityManager`` 在内部使用一个 ``UnitOfWork``，这是
`工作单元模式 <http://martinfowler.com/eaaCatalog/unitOfWork.html>`_
的典型实现，用于跟踪下次调用 ``flush`` 时需要完成的所有事情。
你通常不会直接与一个 ``UnitOfWork`` 交互，而是与 ``EntityManager`` 交互。
