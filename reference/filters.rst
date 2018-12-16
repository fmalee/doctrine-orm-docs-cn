过滤器
=======

.. versionadded:: 2.2

Doctrine2.2具有一个过滤系统，允许开发人员将SQL添加到查询的条件子句中，无论SQL生成的位置如何（例如，从DQL查询，或通过加载相关实体）。
Doctrine 2.2 features a filter system that allows the developer to add SQL to
the conditional clauses of queries, regardless the place where the SQL is
generated (e.g. from a DQL query, or by loading associated entities).

过滤器功能适用于SQL级别。无论一个SQL查询是在一个Persister中、延迟加载期间，在超级延迟的集合中或从DQL。
每次系统迭代所有启用的过滤器时，会添加新的SQL部件作为一个过滤器返回。
The filter functionality works on SQL level. Whether a SQL query is generated
in a Persister, during lazy loading, in extra lazy collections or from DQL.
Each time the system iterates over all the enabled filters, adding a new SQL
part as a filter returns.

通过将SQL添加到查询的条件子句中，筛选器系统筛选出属于SQL结果集级别的实体的行。
这意味着过滤后的实体永远不会被融合（这可能很昂贵）。
By adding SQL to the conditional clauses of queries, the filter system filters
out rows belonging to the entities at the level of the SQL result set. This
means that the filtered entities are never hydrated (which can be expensive).


过滤器类示例
--------------------

在整个文档中，``MyLocaleFilter`` 示例类将用于说明过滤器功能的工作原理。
一个过滤器类必须继承 ``Doctrine\ORM\Query\Filter\SQLFilter`` 基类并实现
``addFilterConstraint`` 方法。该方法接收已过滤实体的 ``ClassMetadata`` 和该实体的SQL表的表别名。

.. note::

    在已联接或单表继承的情况下，你总是被传递继承根的 ``ClassMetadata``。
    这对于避免在应用过滤器时会破坏SQL的这种边缘情况是有必要的。

应通过 ``SQLFilter#setParameter()`` 方法在过滤器对象上设置查询的参数。
只有通过此函数设置的参数才能用于过滤器。``SQLFilter#getParameter()`` 函数负责正确引用参数。

.. code-block:: php

    <?php
    namespace Example;
    use Doctrine\ORM\Mapping\ClassMetaData,
        Doctrine\ORM\Query\Filter\SQLFilter;

    class MyLocaleFilter extends SQLFilter
    {
        public function addFilterConstraint(ClassMetadata $targetEntity, $targetTableAlias)
        {
            // 检查该实体是否实现了LoalAlcess接口
            if (!$targetEntity->reflClass->implementsInterface('LocaleAware')) {
                return "";
            }

            return $targetTableAlias.'.locale = ' . $this->getParameter('locale'); // getParameter会自动应用引用
        }
    }

配置
-------------

过滤器类将如下所示添加到配置中：

.. code-block:: php

    $config->addFilter("locale", "\Doctrine\Tests\ORM\Functional\MyLocaleFilter");

``Configuration#addFilter()`` 方法接受一个过滤器的名称和负责实际过滤的类的名称。

禁用/启用过滤器和设置参数
---------------------------------------------------

可以通过 ``FilterCollection`` 来禁用和启用存储在 ``EntityManager`` 中过滤器。
``FilterCollection#enable($name)`` 方法将检索过滤器对象。你可以在该对象上设置过滤器参数。

.. code-block:: php

    $filter = $em->getFilters()->enable("locale");
    $filter->setParameter('locale', 'en');

    // 禁用它
    $filter = $em->getFilters()->disable("locale");

.. warning::
    禁用和启用过滤器对已管理实体没有影响。如果要在修改一个过滤器或 ``FilterCollection``
    之后刷新或重新加载一个对象，则应清除 ``EntityManager`` 并重新获取你的实体，然后应用新的过滤规则。
