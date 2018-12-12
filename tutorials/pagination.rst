分页
==========

.. versionadded:: 2.2

从2.2版本开始，Doctrine附带了一个用于 ``DQL`` 查询的 ``Paginator``。
它有一个非常简单的API，并实现了 ``Countable`` 和 ``IteratorAggregate`` SPL接口。

.. code-block:: php

    use Doctrine\ORM\Tools\Pagination\Paginator;

    $dql = "SELECT p, c FROM BlogPost p JOIN p.comments c";
    $query = $entityManager->createQuery($dql)
                           ->setFirstResult(0)
                           ->setMaxResults(100);

    $paginator = new Paginator($query, $fetchJoinCollection = true);

    $c = count($paginator);
    foreach ($paginator as $post) {
        echo $post->getHeadline() . "\n";
    }

Doctrine分页查询并不像你在开始时想到的那么简单。
如果你具有使用数据库供应商的“默认” ``LIMIT`` 功能的 **一对多**
或 **多对多** 关联的复杂提取连接(fetch-join)方案，则不足以获得正确的结果。

默认情况下，该分页扩展会执行以下步骤来计算正确的结果：

- 使用 `DISTINCT` 关键字执行一个 ``Count`` 查询。
- 使用 `DISTINCT` 执行一个限制子查询，以在当前页面上查找实体的所有ID。
- 执行一个 ``WHERE IN`` 查询以获取当前页面的所有结果。

仅当你实际获取加入多对多集合时，才需要此行为。
你可以通过将 ``$fetchJoinCollection`` 标志设置为禁用此行为 ``false``;
在这种情况下，只执行 **2** 个而不是所描述的 **3** 个查询。我们希望将来能够自动进行检测。
