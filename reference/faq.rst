常见问题
==========================

.. note::

    此常见问题的解答工作正在进行中。我们将添加许多问题，而不是立即回答它们，只是为了记住经常被问到的问题。
    如果你遇到未解答的问题，请写邮件到邮件列表或加入Freenode IRC的#doctrine频道。

数据库模式
---------------

如何为MySQL表设置 ``charset`` 和 ``collat​​ion`` ？
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

你无法在注释、yml或xml映射文件中设置这些值。
要使数据库使用默认的字符集和排序规则，你应该将MySQL配置为使用默认字符集，或者使用字符集和排序规则的详细信息来创建数据库。
这样，它们就会继承到所有新创建的数据库的表和列中。

实体类
--------------

我访问了一个变量，但它是 ``NULL``，这是什么原因？
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

如果这个变量是一个 ``public`` 变量，那么你违反了实体的一个标准。所有的属性都必须是
``protected`` 或 ``private``，以便代理对象模式能运行。

如何向列添加默认值？
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Doctrine不支持通过SQL中的 ``DEFAULT`` 关键字在列中设置默认值。
但这不是必需的，你可以将类属性用作默认值。然后在插入时使用它们：

.. code-block:: php

    class User
    {
        const STATUS_DISABLED = 0;
        const STATUS_ENABLED = 1;

        private $algorithm = "sha1";
        private $status = self:STATUS_DISABLED;
    }

.

映射
-------

为什么我会在 ``$em->flush()`` 期间获得关于唯一约束失败的异常？
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Doctrine不会检查你是否使用已存在的主键重新添加实体，或者是否将实体添加到集合两次。
如果你知道可能发生唯一约束失败，则必须在调用 ``$em->flush()`` 之前在代码中自行检查这两个约束。

例如在 `Symfony2 <http://www.symfony.com>`_ 中，有一个唯一实体验证器来完成此任务。

对于集合，你可以使用 ``$collection->contains($entity)`` 来检查实体是否已经是此集合的一部分。
对于 ``FETCH = LAZY`` 集合，这将初始化该集合，但是对于
``FETCH = EXTRA_LAZY``，此方法将使用SQL来确定此实体是否已经是集合的一部分。

关联
------------

我获得一个 "A new entity was found through the relationship.." 的 ``InvalidArgumentException``，是什么原因？
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

当标识映射中存在包含对Doctrine不知道的对象的引用的对象时，会抛出此异常。
比如说，你从具有特定ID的数据库中获取“用户”实体，并将一个全新的对象设置为 ``User`` 对象的一个​​关联。
如果你在不让Doctrine知道这个新对象的情况下调用 ``EntityManager#flush()`` ，你将看到此异常。
This exception is thrown during ``EntityManager#flush()`` when there exists an object in the identity map
that contains a reference to an object that Doctrine does not know about. Say for example you grab
a "User"-entity from the database with a specific id and set a completely new object into one of the associations
of the User object. If you then call ``EntityManager#flush()`` without letting Doctrine know about
this new object using ``EntityManager#persist($newObject)`` you will see this exception.

你可以通过以下方式解决此异常：

* 在新对象上调用 ``EntityManager#persist($newObject)``
* 在包含新对象的关联上使用 ``cascade=persist``

如何过滤一个关联？
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

你无法原生的过滤2.0和2.1中的关联。你应该使用DQL查询来查询已过滤的实体集。

我在多对一集合上调用 ``clear()``，但实体未被删除
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

这是一种预期的行为，与Doctrine的从属方/拥有方处理有关。
根据定义，一对多关联是在从属方的，这意味着Doctrine不会识别对它的更改。

如果要执行与 ``clear`` 操作等效的操作，则必须迭代集合并将拥有方的多对一引用设置为 ``NULL``，以便从集合中分离所有实体。
这将在数​​据库上触发相应的 ``UPDATE`` 语句。

如何将列添加到多对多的表中？
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

多对多关联仅支持表定义中的外键，要使用包含额外列的多对多表，必须使用将外键作为主键的功能(在Doctrine 2.1中引入)。

更多信息，请参阅 :doc:`有关复合主键的教程<../tutorials/composite-primary-keys>`。

我如何对提取联接集合进行分页？
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

如果要发出一个DQL语句来提取一个集合，则无法使用 ``LIMIT`` 语句（或供应商的等效语句）来轻松迭代此集合。

Doctrine并不提供开箱即用的解决方案，但有几个扩展可供使用：

* `DoctrineExtensions <http://github.com/beberlei/DoctrineExtensions>`_
* `Pagerfanta <http://github.com/whiteoctober/pagerfanta>`_

为什么分页无法与提取链接一起工作？
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Doctrine中的分页使用 ``LIMIT`` 子句（或供应商的等效语句）来限制结果。
但是，当提取链接时，由于与一对多或多对多关联的联接将行数乘以关联实体的数量，因此未返回正确数量的结果。

有关此任务的解决方案，请参阅上一个问题。

继承
-----------

我可以在Doctrine2中使用继承吗？
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

是的，你可以在Doctrine2中使用单表或联接表继承。

有关详细信息，请参阅有关 :doc:`继承映射 <inheritance-mapping>` 的文档。

为什么Doctrine不为我的继承层级创建代理对象？
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

如果将多对一或一对一关联目标实体设置为继承层级的任何父类，则Doctrine不知道外部实际使用的PHP类。
要找到它，必须执行SQL查询以在数据库中查找此信息。
If you set a many-to-one or one-to-one association target-entity to any parent class of
an inheritance hierarchy Doctrine does not know what PHP class the foreign is actually of.
To find this out it has to execute a SQL query to look this information up in the database.

实体生成器
---------------

为什么实体生成器不能做 ``X``？
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

实体生成器不是一个完整的可以解决所有任务的代码生成器。
代码生成不再是Doctrine2中的一流优先级（与Doctrine1相比）。EntityGenerator应该启动你，但不是100％。
The EntityGenerator is not a full fledged code-generator that solves all tasks. Code-Generation
is not a first-class priority in Doctrine 2 anymore (compared to Doctrine 1). The EntityGenerator
is supposed to kick-start you, but not towards 100%.

为什么 ``EntityGenerator`` 不能正确生成继承？
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

仅从鉴别器映射的细节来看，``EntityGenerator`` 无法猜测继承层级。
这就是为什么继承实体的生成不能完全发挥作用的原因。你必须调整一些额外的代码才能使之正常工作。

性能
-----------

为什么每次获取具有一对一关系的实体时都会执行额外的SQL查询？
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

如果Doctrine检测到你正在获取一个从属方的一对一关联，则必须执行一个额外查询来加载此对象，因为它无法知道是否有此类对象（设置为 ``null``），或者是否应设置代理以及该代理有哪些ID。

为了解决这个问题，目前必须执行查询以找出该信息。

Doctrine查询语言
-----------------------

什么是DQL？
~~~~~~~~~~~~

DQL即Doctrine查询语言，这是一种看起来像SQL的查询语言，但在使用Doctrine时有一些重要的好处：

-  它使用类名和字段来代替表和列，以分离后端和对象模型之间的关注点。
-  它利用已定义的元数据在写入时提供一系列快捷方式。例如，你不必指定连接的 ``ON`` 子句，因为Doctrine已经知道它们。
-  它添加了一些与对象管理相关的功能，并将它们转换为SQL。

它当然也有一些缺点：

-  语法与SQL略有不同，因此你必须学习并记住这些差异。
-  要独立于供应商，它只能实现所有现有SQL方言的一个子集。除非明确实现，否则不能通过DQL使用供应商特定的功能和优化。
-  对于某些DQL构造，使用了已知在MySQL中较慢的子选择。For some DQL constructs subselects are used which are known to be slow in MySQL.

我可以在DQL中按函数排序（例如 ``ORDER BY RAND()``）吗？
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

不，不支持在DQL中按函数排序。如果你需要此功能，则应使用原生查询或提出其他解决方案。
作为旁注：使用 ``ORDER BY RAND()`` 进行排序从1000行开始是非常缓慢的。

一个查询失败了，我该如何调试？
----------------------------------

首先，如果你使用的是 ``QueryBuilder``，则可以使用`` $queryBuilder->getDQL()`` 来获取此查询的DQL字符串。
你可以通过调用 ``$query->getSQL()`` 来从Query实例中获取相应的SQL。

.. code-block:: php

    <?php
    $dql = "SELECT u FROM User u";
    $query = $entityManager->createQuery($dql);
    var_dump($query->getSQL());

    $qb = $entityManager->createQueryBuilder();
    $qb->select('u')->from('User', 'u');
    var_dump($qb->getDQL());
