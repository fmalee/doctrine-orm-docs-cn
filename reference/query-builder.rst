查询构建器
================

一个 ``QueryBuilder`` 提供了一个用于在几个步骤中有条件地构建DQL查询的API。

它提供了一组能够以编程的方式来构建查询的类和方法，并提供了一个流式的API。
这意味着，你可以在一种方法之间转换到另一种方法，或者只是选择一种优选方法。

构造一个新的QueryBuilder对象
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

与构建一个普通的 ``Query`` 的方式相同，你可以构建一个 ``QueryBuilder`` 对象。
以下是如何构建一个 ``QueryBuilder`` 对象的示例：

.. code-block:: php

    // $em 为 EntityManager 的实例

    // 示例1: 创建一个 QueryBuilder 实例
    $qb = $em->createQueryBuilder();

``QueryBuilder`` 的一个实例有几个信息方法。一个很好的例子是检查它是什么类型的 ``QueryBuilder`` 对象。

.. code-block:: php

    // $qb 为 QueryBuilder 的实例

    // 示例2: 获取 QueryBuilder 的类型
    echo $qb->getType(); // 打印: 0

目前 ``getType()`` 的返回值有 *三* 种可能：


-  ``QueryBuilder::SELECT``, 返回值为 ``0``
-  ``QueryBuilder::DELETE``, 返回值为 ``1``
-  ``QueryBuilder::UPDATE``, 返回值为 ``2``

你的 ``DQL`` 完成构建后，可以检索已关联 ``EntityManager`` 的当前
``QueryBuilder``、``DQL`` 甚至一个 ``Query`` 对象。
It is possible to retrieve the associated ``EntityManager`` of the
current ``QueryBuilder``, its DQL and also a ``Query`` object when
you finish building your DQL.

.. code-block:: php

    // $qb 为 QueryBuilder 的实例

    // 示例3: 检索已关联的EntityManager
    $em = $qb->getEntityManager();

    // 示例4: 检索 QueryBuilder 中定义的 DQL 字符串
    $dql = $qb->getDql();

    // 示例5: 使用已处理的 DQL 来检索已关联的 Query 对象
    $q = $qb->getQuery();

在内部，``QueryBuilder`` 使用一个 *DQL缓存* 来提高性能。
任何可能影响生成的DQL的更改实际上会将 ``QueryBuilder`` 的状态修改为我们称为 ``STATE_DIRTY`` 的阶段。
一个 ``QueryBuilder`` 可以处于两种不同的状态：

-  ``QueryBuilder::STATE_CLEAN``, 这意味着自上次检索以来DQL没有被更改，或者自实例化以来没有添加任何内容。
-  ``QueryBuilder::STATE_DIRTY``, 这意味着 *必须* （并将）在下次检索时处理该DQL查询

使用查询构建器
~~~~~~~~~~~~~~~~~~~~~~~~~

高级API方法
^^^^^^^^^^^^^^^^^^^^^^

为了简化在Doctrine中构建一个查询的方式，你可以利用辅助方法。
对于所有基本代码，有一组有用的方法可以简化程序员的生活。
为了说明如何使用它们，下面是使用 ``QueryBuilder`` 辅助方法来重写的相同示例6：
To simplify even more the way you build a query in Doctrine, you can take
advantage of Helper methods. For all base code, there is a set of
useful methods to simplify a programmer's life. To illustrate how
to work with them, here is the same example 6 re-written using
``QueryBuilder`` helper methods:

.. code-block:: php

    // $qb 为 QueryBuilder 的实例

    $qb->select('u')
       ->from('User', 'u')
       ->where('u.id = ?1')
       ->orderBy('u.name', 'ASC');

``QueryBuilder`` 辅助方法被认为是构建DQL查询的标准方法。
虽然支持基于字符串的查询，但应避免使用它。我们非常鼓励你使用 ``$qb->expr()->*`` 方法。
这是一个转换后的示例8，用来建议构建查询的标准方式：

.. code-block:: php

    // $qb 为 QueryBuilder 的实例

    $qb->select(array('u')) // 字符串 'u' 在内部转换为数组
       ->from('User', 'u')
       ->where($qb->expr()->orX(
           $qb->expr()->eq('u.id', '?1'),
           $qb->expr()->like('u.nickname', '?2')
       ))
       ->orderBy('u.surname', 'ASC');

以下是在 ``QueryBuilder`` 中可用的辅助方法的完整列表：

.. code-block:: php

    class QueryBuilder
    {
        // 示例 - $qb->select('u')
        // 示例 - $qb->select(array('u', 'p'))
        // 示例 - $qb->select($qb->expr()->select('u', 'p'))
        public function select($select = null);

        // 备注: addSelect() 不会覆盖之前的 select() 调用
        //
        // 示例 - $qb->select('u');
        //              ->addSelect('p.area_code');
        public function addSelect($select = null);

        // 示例 - $qb->delete('User', 'u')
        public function delete($delete = null, $alias = null);

        // 示例 - $qb->update('Group', 'g')
        public function update($update = null, $alias = null);

        // 示例 - $qb->set('u.firstName', $qb->expr()->literal('Arnold'))
        // 示例 - $qb->set('u.numChilds', 'u.numChilds + ?1')
        // 示例 - $qb->set('u.numChilds', $qb->expr()->sum('u.numChilds', '?1'))
        public function set($key, $value);

        // 示例 - $qb->from('Phonenumber', 'p')
        // 示例 - $qb->from('Phonenumber', 'p', 'p.id')
        public function from($from, $alias, $indexBy = null);

        // 示例 - $qb->join('u.Group', 'g', Expr\Join::WITH, $qb->expr()->eq('u.status_id', '?1'))
        // 示例 - $qb->join('u.Group', 'g', 'WITH', 'u.status = ?1')
        // 示例 - $qb->join('u.Group', 'g', 'WITH', 'u.status = ?1', 'g.id')
        public function join($join, $alias, $conditionType = null, $condition = null, $indexBy = null);

        // 示例 - $qb->innerJoin('u.Group', 'g', Expr\Join::WITH, $qb->expr()->eq('u.status_id', '?1'))
        // 示例 - $qb->innerJoin('u.Group', 'g', 'WITH', 'u.status = ?1')
        // 示例 - $qb->innerJoin('u.Group', 'g', 'WITH', 'u.status = ?1', 'g.id')
        public function innerJoin($join, $alias, $conditionType = null, $condition = null, $indexBy = null);

        // 示例 - $qb->leftJoin('u.Phonenumbers', 'p', Expr\Join::WITH, $qb->expr()->eq('p.area_code', 55))
        // 示例 - $qb->leftJoin('u.Phonenumbers', 'p', 'WITH', 'p.area_code = 55')
        // 示例 - $qb->leftJoin('u.Phonenumbers', 'p', 'WITH', 'p.area_code = 55', 'p.id')
        public function leftJoin($join, $alias, $conditionType = null, $condition = null, $indexBy = null);

        // 备注: where() 会覆盖所有先前设置的条件
        //
        // 示例 - $qb->where('u.firstName = ?1', $qb->expr()->eq('u.surname', '?2'))
        // 示例 - $qb->where($qb->expr()->andX($qb->expr()->eq('u.firstName', '?1'), $qb->expr()->eq('u.surname', '?2')))
        // 示例 - $qb->where('u.firstName = ?1 AND u.surname = ?2')
        public function where($where);

        // 备注: andWhere() 可以直接使用，而无需在之前添加任何 where()
        //
        // 示例 - $qb->andWhere($qb->expr()->orX($qb->expr()->lte('u.age', 40), 'u.numChild = 0'))
        public function andWhere($where);

        // 示例 - $qb->orWhere($qb->expr()->between('u.id', 1, 10));
        public function orWhere($where);

        // NOTE: -> groupBy() overrides all previously set grouping conditions
        //
        // 示例 - $qb->groupBy('u.id')
        public function groupBy($groupBy);

        // 示例 - $qb->addGroupBy('g.name')
        public function addGroupBy($groupBy);

        // 备注: having() 覆盖之前设置的所有条件
        //
        // 示例 - $qb->having('u.salary >= ?1')
        // 示例 - $qb->having($qb->expr()->gte('u.salary', '?1'))
        public function having($having);

        // 示例 - $qb->andHaving($qb->expr()->gt($qb->expr()->count('u.numChild'), 0))
        public function andHaving($having);

        // 示例 - $qb->orHaving($qb->expr()->lte('g.managerLevel', '100'))
        public function orHaving($having);

        // 备注: orderBy() 会覆盖所有先前设置的排序条件
        //
        // 示例 - $qb->orderBy('u.surname', 'DESC')
        public function orderBy($sort, $order = null);

        // 示例 - $qb->addOrderBy('u.firstName')
        public function addOrderBy($sort, $order = null); // Default $order = 'ASC'
    }

将参数绑定到查询
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Doctrine支持将参数动态绑定到查询，类似于准备（preparing）查询。
你可以使用 *字符串* 和 *数字* 作为占位符，但两者的语法略有不同。
此外，你必须做出选择：不允许混合使用这两种样式。绑定参数可以简单地实现如下：

.. code-block:: php

    // $qb 为 QueryBuilder 的实例

    $qb->select('u')
       ->from('User', 'u')
       ->where('u.id = ?1')
       ->orderBy('u.name', 'ASC')
       ->setParameter(1, 100); // 设置 ?1 为 100, 因此我们将获取一个 `u.id = 100` 的用户

你不必强制枚举占位符，因为有备用语法可用：

.. code-block:: php

    // $qb 为 QueryBuilder 的实例

    $qb->select('u')
       ->from('User', 'u')
       ->where('u.id = :identifier')
       ->orderBy('u.name', 'ASC')
       ->setParameter('identifier', 100); // 设置 :identifier 为 100, 因此我们将获取一个 `u.id = 100` 的用户

请注意，**数字占位符** 以一个 ``?`` 开头，后跟一个数字，而 **命名占位符** 以一个 ``:`` 开头，后跟一个字符串。

调用 ``setParameter()`` 时会自动推断你将哪种类型设置为值。
这适用于整数、字符串/整数数组、``DateTime`` 实例以及已管理实体。
如果要显式设置一个类型，可以显式调用 ``setParameter()``
的第三个参数，它接受一个 ``PDO`` 类型或一个 ``DBAL`` 类型名称进行转换。

如果你有几个参数要绑定到你的查询，你还可以使用 ``setParameters()`` 而不是 ``setParameter()``，请使用以下语法：

.. code-block:: php

    // $qb 为 QueryBuilder 的实例

    // 在这里查询...
    $qb->setParameters(array(1 => 'value for ?1', 2 => 'value for ?2'));

获取已经绑定的参数很简单 - 只需将上面提到的语法应用到 ``getParameter()`` 或 ``getParameters()``：

.. code-block:: php

    // $qb 为 QueryBuilder 的实例

    // 查看上面的示例
    $params = $qb->getParameters();
    // $params 是 \Doctrine\Common\Collections\ArrayCollection 的实例

    // 等同于
    $param = $qb->getParameter(1);
    // $param 是 \Doctrine\ORM\Query\Parameter 的实例

注意：如果你尝试获取一个尚未绑定的参数，则 ``getParameter()`` 只返回 ``NULL``。

一个查询参数的API是：

.. code-block:: php

    namespace Doctrine\ORM\Query;

    class Parameter
    {
        public function getName();
        public function getValue();
        public function getType();
        public function setValue($value, $type = null);
    }

限制查询结果
^^^^^^^^^^^^^^^^^^^

为了限制一个查询结果，查询构建器有一些与可以从 ``EntityManager#createQuery()``
中检索的 ``Query`` 对象相同的方法。

.. code-block:: php

    // $qb 为 QueryBuilder 的实例
    $offset = (int)$_GET['offset'];
    $limit = (int)$_GET['limit'];

    $qb->add('select', 'u')
       ->add('from', 'User u')
       ->add('orderBy', 'u.name ASC')
       ->setFirstResult( $offset )
       ->setMaxResults( $limit );

执行查询
^^^^^^^^^^^^^^^^^

``QueryBuilder`` 只是一个构建器对象 - 它无法实际执行 ``Query``。
此外，无法在 ``QueryBuilder`` 本身上设置一组参数，如查询提示。
这就是你始终必须将一个查询构建器实例转换为 ``Query`` 对象的原因：

.. code-block:: php

    // $qb 为 QueryBuilder 的实例
    $query = $qb->getQuery();

    // 设置其他查询选项
    $query->setQueryHint('foo', 'bar');
    $query->useResultCache('my_cache_id');

    // 执行查询
    $result = $query->getResult();
    $single = $query->getSingleResult();
    $array = $query->getArrayResult();
    $scalar = $query->getScalarResult();
    $singleScalar = $query->getSingleScalarResult();

``Expr`` 类
^^^^^^^^^^^^^^

为了解决 ``add()`` 方法可能导致的一些问题，Doctrine创建了一个可以将其视为构建表达式的一个辅助方法的类。
该类名称为 ``Expr``，它提供了一组有用的方法来帮助构建表达式：

.. code-block:: php

    // $qb 为 QueryBuilder 的实例

    // 示例8: QueryBuilder port of:
    // "SELECT u FROM User u WHERE u.id = ? OR u.nickname LIKE ? ORDER BY u.name ASC" using Expr class
    $qb->add('select', new Expr\Select(array('u')))
       ->add('from', new Expr\From('User', 'u'))
       ->add('where', $qb->expr()->orX(
           $qb->expr()->eq('u.id', '?1'),
           $qb->expr()->like('u.nickname', '?2')
       ))
       ->add('orderBy', new Expr\OrderBy('u.name', 'ASC'));

虽然它听起来仍然很复杂，但编程的创建条件的能力是 ``Expr`` 的主要功能。这里是可用的辅助方法的完整列表：

.. code-block:: php

    class Expr
    {
        /** 条件对象 **/

        // 示例 - $qb->expr()->andX($cond1 [, $condN])->add(...)->...
        public function andX($x = null); // 返回 Expr\AndX 实例

        // 示例 - $qb->expr()->orX($cond1 [, $condN])->add(...)->...
        public function orX($x = null); // 返回 Expr\OrX 实例


        /** 比较对象 **/

        // 示例 - $qb->expr()->eq('u.id', '?1') => u.id = ?1
        public function eq($x, $y); // 返回 Expr\Comparison 实例

        // 示例 - $qb->expr()->neq('u.id', '?1') => u.id <> ?1
        public function neq($x, $y); // 返回 Expr\Comparison 实例

        // 示例 - $qb->expr()->lt('u.id', '?1') => u.id < ?1
        public function lt($x, $y); // 返回 Expr\Comparison 实例

        // 示例 - $qb->expr()->lte('u.id', '?1') => u.id <= ?1
        public function lte($x, $y); // 返回 Expr\Comparison 实例

        // 示例 - $qb->expr()->gt('u.id', '?1') => u.id > ?1
        public function gt($x, $y); // 返回 Expr\Comparison 实例

        // 示例 - $qb->expr()->gte('u.id', '?1') => u.id >= ?1
        public function gte($x, $y); // 返回 Expr\Comparison 实例

        // 示例 - $qb->expr()->isNull('u.id') => u.id IS NULL
        public function isNull($x); // 返回 string

        // 示例 - $qb->expr()->isNotNull('u.id') => u.id IS NOT NULL
        public function isNotNull($x); // 返回 string


        /** 算术对象 **/

        // 示例 - $qb->expr()->prod('u.id', '2') => u.id * 2
        public function prod($x, $y); // 返回 Expr\Math 实例

        // 示例 - $qb->expr()->diff('u.id', '2') => u.id - 2
        public function diff($x, $y); // 返回 Expr\Math 实例

        // 示例 - $qb->expr()->sum('u.id', '2') => u.id + 2
        public function sum($x, $y); // 返回 Expr\Math 实例

        // 示例 - $qb->expr()->quot('u.id', '2') => u.id / 2
        public function quot($x, $y); // 返回 Expr\Math 实例


        /** 伪函数对象 **/

        // 示例 - $qb->expr()->exists($qb2->getDql())
        public function exists($subquery); // 返回 Expr\Func 实例

        // 示例 - $qb->expr()->all($qb2->getDql())
        public function all($subquery); // 返回 Expr\Func 实例

        // 示例 - $qb->expr()->some($qb2->getDql())
        public function some($subquery); // 返回 Expr\Func 实例

        // 示例 - $qb->expr()->any($qb2->getDql())
        public function any($subquery); // 返回 Expr\Func 实例

        // 示例 - $qb->expr()->not($qb->expr()->eq('u.id', '?1'))
        public function not($restriction); // 返回 Expr\Func 实例

        // 示例 - $qb->expr()->in('u.id', array(1, 2, 3))
        // 请确保你为使用类似 $qb->expr()->in('value', array('stringvalue')) 的方法
        // 要不然 Doctrine 会抛出一个异常.
        // 相反，使用 $qb->expr()->in('value', array('?1')) 并且绑定你的参数为 ?1 (查看之前的章节)
        public function in($x, $y); // 返回 Expr\Func 实例

        // 示例 - $qb->expr()->notIn('u.id', '2')
        public function notIn($x, $y); // 返回 Expr\Func 实例

        // 示例 - $qb->expr()->like('u.firstname', $qb->expr()->literal('Gui%'))
        public function like($x, $y); // 返回 Expr\Comparison 实例

        // 示例 - $qb->expr()->notLike('u.firstname', $qb->expr()->literal('Gui%'))
        public function notLike($x, $y); // 返回 Expr\Comparison 实例

        // 示例 - $qb->expr()->between('u.id', '1', '10')
        public function between($val, $x, $y); // 返回 Expr\Func 实例


        /** 函数对象 **/

        // 示例 - $qb->expr()->trim('u.firstname')
        public function trim($x); // 返回 Expr\Func 实例

        // 示例 - $qb->expr()->concat('u.firstname', $qb->expr()->concat($qb->expr()->literal(' '), 'u.lastname'))
        public function concat($x, $y); // 返回 Expr\Func 实例

        // 示例 - $qb->expr()->substring('u.firstname', 0, 1)
        public function substring($x, $from, $len); // 返回 Expr\Func 实例

        // 示例 - $qb->expr()->lower('u.firstname')
        public function lower($x); // 返回 Expr\Func 实例

        // 示例 - $qb->expr()->upper('u.firstname')
        public function upper($x); // 返回 Expr\Func 实例

        // 示例 - $qb->expr()->length('u.firstname')
        public function length($x); // 返回 Expr\Func 实例

        // 示例 - $qb->expr()->avg('u.age')
        public function avg($x); // 返回 Expr\Func 实例

        // 示例 - $qb->expr()->max('u.age')
        public function max($x); // 返回 Expr\Func 实例

        // 示例 - $qb->expr()->min('u.age')
        public function min($x); // 返回 Expr\Func 实例

        // 示例 - $qb->expr()->abs('u.currentBalance')
        public function abs($x); // 返回 Expr\Func 实例

        // 示例 - $qb->expr()->sqrt('u.currentBalance')
        public function sqrt($x); // 返回 Expr\Func 实例

        // 示例 - $qb->expr()->count('u.firstname')
        public function count($x); // 返回 Expr\Func 实例

        // 示例 - $qb->expr()->countDistinct('u.surname')
        public function countDistinct($x); // 返回 Expr\Func 实例
    }

向查询添加Criteria
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

你还可以使用 ``addCriteria`` 命令将 :ref:`Criteria <filtering-collections>` 添加到 ``QueryBuilder``：

.. code-block:: php

    <?php
    use Doctrine\Common\Collections\Criteria;
    // ...

    $criteria = Criteria::create()
        ->orderBy(['firstName', 'ASC']);

    // $qb 为 QueryBuilder 实例
    $qb->addCriteria($criteria);
    // then execute your query like normal

低级API
^^^^^^^^^^^^^

现在我们将描述创建查询的低级方法。在此级别工作以进行优化可能很有用，但大多数情况下，最好在更高的抽象级别上工作。

``QueryBuilder`` 的所有辅助方法实际上都依赖于单个方法：``add()``。
此方法负责构建每个 ``DQL``。这需要 *三* 个参数：``$dqlPartName``、``$dqlPart`` 以及
``$append`` (default=false)。

-  ``$dqlPartName``: 应该将 ``$dqlPart`` 放在哪里。可能的值：
   ``select``、``from``、``where``、``groupBy``、``having``、``orderBy``
-  ``$dqlPart``: 应该在 ``$dqlPartName`` 中放置什么。接受一个字符串或任何
   ``Doctrine\ORM\Query\Expr\*`` 实例。
-  ``$append``: 可选标志（默认值为 ``FALSE``），``$dqlPart``
   是否应该覆盖所有先前在 ``$dqlPartName`` 中定义的项（对 ``where`` 和 ``having``
   DQL查询部分没有影响，因为它总是覆盖所有先前定义的项）

.. code-block:: php

    // $qb 为 QueryBuilder 实例

    // 示例6: 如何定义:
    // "SELECT u FROM User u WHERE u.id = ? ORDER BY u.name ASC"
    // 使用 QueryBuilder 的字符串支持
    $qb->add('select', 'u')
       ->add('from', 'User u')
       ->add('where', 'u.id = ?1')
       ->add('orderBy', 'u.name ASC');

``Expr\*`` 类
^^^^^^^^^^^^^^

当你使用字符串调用 ``add()`` 时，它会在内部求值为 ``Doctrine\ORM\Query\Expr\Expr\*`` 类的实例。
以下是使用 ``Doctrine\ORM\Query\Expr\Expr\*`` 类编写与示例6相同的查询：

.. code-block:: php

   <?php
   // $qb 为 QueryBuilder 实例

   // 示例7: 如何定义:
   // "SELECT u FROM User u WHERE u.id = ? ORDER BY u.name ASC"
   // 使用 QueryBuilder 以及 Expr\* 实例
   $qb->add('select', new Expr\Select(array('u')))
      ->add('from', new Expr\From('User', 'u'))
      ->add('where', new Expr\Comparison('u.id', '=', '?1'))
      ->add('orderBy', new Expr\OrderBy('u.name', 'ASC'));
