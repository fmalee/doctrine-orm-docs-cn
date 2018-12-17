有序关联
-----------------------------

有些用例需要在从数据库中检索时对集合进行排序。
在用户空间中，只要你最初没有将具有关联的实体保存到数据库中，就可以执行此操作。
要从数据库中检索一个已排序的集合，你可以将 ``@OrderBy``
注释与一个集合一起使用，用来指定一个会附加到具有此集合的所有查询的DQL片段。

除了添加到任何 ``@OneToMany`` 或 ``@ManyToMany`` 注释之外，你还可以通过以下方式来指定 ``@OrderBy``：

.. configuration-block::

    .. code-block:: php

        <?php
        /** @Entity **/
        class User
        {
            // ...

            /**
             * @ManyToMany(targetEntity="Group")
             * @OrderBy({"name" = "ASC"})
             **/
            private $groups;
        }

    .. code-block:: xml

        <doctrine-mapping>
            <entity name="User">
                <many-to-many field="groups" target-entity="Group">
                    <order-by>
                        <order-by-field name="name" direction="ASC" />
                    </order-by>
                </many-to-many>
            </entity>
        </doctrine-mapping>

    .. code-block:: yaml

        User:
          type: entity
          manyToMany:
            groups:
              orderBy: { 'name': 'ASC' }
              targetEntity: Group
              joinTable:
                name: users_groups
                joinColumns:
                  user_id:
                    referencedColumnName: id
                inverseJoinColumns:
                  group_id:
                    referencedColumnName: id

``@OrderBy`` 中的DQL代码段只允许由未限定的、未引用的字段名称和一个可选的
``ASC`` / ``DESC`` 位置语句组成。多个字段可用逗号（``,``）分隔。
引用的字段名称必须存在于 ``@ManyToMany`` 或 ``@OneToMany`` 注释的 ``targetEntity`` 类中。

该功能的语义可以描述如下：

-  ``@OrderBy`` 充当给定字段的一个隐式 ``ORDER BY`` 子句，该子句会附加到所有显式给定的 ``ORDER BY`` 项中。
-  始终以有序的方式来检索所有拥有有序类型的集合。
-  为了保持较低的数据库影响，如果在DQL查询中对集合进行提取联接，则这些隐式 ``ORDER BY`` 项仅添加到一个DQL查询中。

鉴于我们之前定义的示例，以下内容不会添加 ``ORDER BY``，因为 ``g`` 不是提取联接：

.. code-block:: sql

    SELECT u FROM User u JOIN u.groups g WHERE SIZE(g) > 10

但是以下内容：

.. code-block:: sql

    SELECT u, g FROM User u JOIN u.groups g WHERE u.id = 10

......将在内部重写为：

.. code-block:: sql

    SELECT u, g FROM User u JOIN u.groups g WHERE u.id = 10 ORDER BY g.name ASC

你可以使用显式的DQL ``ORDER BY`` 来逆向排序：

.. code-block:: sql

    SELECT u, g FROM User u JOIN u.groups g WHERE u.id = 10 ORDER BY g.name DESC

......将在内部被重写为：

.. code-block:: sql

    SELECT u, g FROM User u JOIN u.groups g WHERE u.id = 10 ORDER BY g.name DESC, g.name ASC
