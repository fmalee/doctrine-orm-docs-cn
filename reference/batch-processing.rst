批量处理
================

本章介绍如何以高效的方式使用Doctrine完成批量插入、更新和删除。
批量操作的主要问题通常不是内存耗尽，尤其这里提供的策略提供了一些帮助。

.. warning::

    一个ORM工具主要不适合大规模插入、更新或删除。
    每个RDBMS都有自己最有效的处理此类操作的方法，如果下面列出的选项不足以满足你的需要，我们建议你使用适用于这些批量操作的特定RDBMS工具。

批量插入
------------

Doctrine中的批量插入最好利用 ``EntityManager`` 的事务后写行为进行分批执行。
以下代码展示了插入批处理数量为20的10000个对象的示例。
你可能需要尝试不同的批处理数量以找到最适合你的数量。
更大的批量意味着更多的准备语句在内部重用，但也意味着在 ``flush`` 期间要进行更多的工作。

.. code-block:: php

    $batchSize = 20;
    for ($i = 1; $i <= 10000; ++$i) {
        $user = new CmsUser;
        $user->setStatus('user');
        $user->setUsername('user' . $i);
        $user->setName('Mr.Smith-' . $i);
        $em->persist($user);
        if (($i % $batchSize) === 0) {
            $em->flush();
            $em->clear(); // 从Doctrine中分离所有对象！
        }
    }
    $em->flush(); // 持久化对象当不为批量处理
    $em->clear();

批量更新
------------

使用Doctrine进行批量更新有 *两* 种可能性。

DQL UPDATE
~~~~~~~~~~

最有效的批量更新方法是使用 ``DQL UPDATE`` 查询。例如：

.. code-block:: php

    $q = $em->createQuery('update MyProject\Model\Manager m set m.salary = m.salary * 0.9');
    $numUpdated = $q->execute();

迭代结果
~~~~~~~~~~~~~~~~~

批量更新的替代解决方案是使用 ``Query#iterate()`` 工具逐步迭代查询结果，而不是立即将整个结果加载到内存中。
以下示例展示了如何执行此操作，就是将迭代与已用于批量插入的批处理策略相结合：

.. code-block:: php

    $batchSize = 20;
    $i = 0;
    $q = $em->createQuery('select u from MyProject\Model\User u');
    $iterableResult = $q->iterate();
    foreach ($iterableResult as $row) {
        $user = $row[0];
        $user->increaseCredit();
        $user->calculateNewBonuses();
        if (($i % $batchSize) === 0) {
            $em->flush(); // 执行所有更新
            $em->clear(); // 从Doctrine中分离所有对象！
        }
        ++$i;
    }
    $em->flush();

.. note::

    对于获取联接(fetch-join)一个已集合值(collection-valued)的关联的查询，无法对迭代结果进行迭代。
    此类SQL结果集的性质不适合增量融合(hydration)。

.. note::

    结果可能被数据库的客户端/连接完全缓冲，它们分配不可见的额外内存给PHP进程。
    对于大型集合，这可能很容易在无明显原因的情况下终止该进程。

批量删除
------------

使用Doctrine进行批量删除有 *两* 种可能性。你可以发出单个 ``DQL DELETE`` 查询，也可以迭代结果，一次删除一个。

DQL DELETE
~~~~~~~~~~

最有效的批量删除方法是使用 ``DQL DELETE`` 查询。

示例:

.. code-block:: php

    $q = $em->createQuery('delete from MyProject\Model\Manager m where m.salary > 100000');
    $numDeleted = $q->execute();

迭代结果
~~~~~~~~~~~~~~~~~

批量删除的替代解决方案是使用 ``Query#iterate()`` 工具逐步迭代查询结果，而不是立即将整个结果加载到内存中。
以下示例展示了如何执行此操作：

.. code-block:: php

    $batchSize = 20;
    $i = 0;
    $q = $em->createQuery('select u from MyProject\Model\User u');
    $iterableResult = $q->iterate();
    while (($row = $iterableResult->next()) !== false) {
        $em->remove($row[0]);
        if (($i % $batchSize) === 0) {
            $em->flush(); // 执行所有删除。
            $em->clear(); // 从Doctrine中分离所有对象！
        }
        ++$i;
    }
    $em->flush();

.. note::

    对于获取联接一个已集合值的关联的查询，无法对迭代结果进行迭代。
    此类SQL结果集的性质不适合增量融合。


为数据处理迭代大型结果
-------------------------------------------

你可以使用 ``iterate()`` 方法迭代一个大型结果，而不是试图 ``UPDATE`` 或 ``DELETE``。
从 ``$query->iterate()`` 返回的 ``IterableResult`` 实例实现了 ``Iterator``
接口，因此你可以使用以下方法处理大型结果而不会出现内存问题：

.. code-block:: php

    $q = $this->_em->createQuery('select u from MyProject\Model\User u');
    $iterableResult = $q->iterate();
    foreach ($iterableResult as $row) {
        // 对行中的数据进行处理，$row[0] 始终是该对象

        // 从Doctrine脱离，以便立即回收垃圾
        $this->_em->detach($row[0]);
    }

.. note::

    对于获取联接一个已集合值的关联的查询，无法对迭代结果进行迭代。
    此类SQL结果集的性质不适合增量融合。
