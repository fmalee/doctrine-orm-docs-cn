超级延迟关联
=======================

.. versionadded:: 2.1

在许多情况下，实体之间的关联可能变得非常大。
例如在像博客这样的简单场景中，如果帖子可以评论，你总是要假设一篇文章吸引了数百条评论。
在Doctrine 2.0中，如果你访问了一个关联，它将始终被完全加载到内存中。
如果你的关联包含数百或数千个实体，就可能会导致严重的性能问题。

使用Doctrine 2.1，为关联引入了一个名为 **超级延迟** 的功能。
默认情况下，关联标记为 **延迟**，这意味着一个关联的整个集合对象会在第一次访问时填充。
如果将关联标记为 *超级延迟*，则可以在不触发集合的完整加载的情况下调用集合上的以下方法：

-  ``Collection#contains($entity)``
-  ``Collection#containsKey($key)`` （可与Doctrine 2.5一起使用）
-  ``Collection#count()``
-  ``Collection#get($key)`` （可与Doctrine 2.4一起使用）
-  ``Collection#slice($offset, $length = null)``

对于上述每种方法，适用以下语义：

-  对于每个调用，如果集合尚未加载，则对数据库发出一个直接的 ``SELECT`` 语句。
-  对于每个调用，如果集合已经加载，则回退到延迟集合的默认功能。不再执行额外的 ``SELECT`` 语句。

此外，即使使用Doctrine 2.0，以下方法也不会触发集合的加载：

-  ``Collection#add($entity)``
-  ``Collection#offsetSet($key, $entity)`` - 没有特定 ``$coll[] = $entity`` 键的 ``ArrayAccess``，在设置特定
   ``$coll[0] = $entity`` 键时不起作用。

配合超级延迟集合，你现在不仅可以添加实体到大型集合，而且容易使用的 ``count`` 和 ``slice`` 组合进行分页。

启用超级延迟关联
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

该映射配置很简单。不是使用 ``fetch="LAZY"`` 默认值，而是切换到 ``EXTRA_LAZY``，如下面的示例所示：

.. configuration-block::

    .. code-block:: php

        <?php
        namespace Doctrine\Tests\Models\CMS;

        /**
         * @Entity
         */
        class CmsGroup
        {
            /**
             * @ManyToMany(targetEntity="CmsUser", mappedBy="groups", fetch="EXTRA_LAZY")
             */
            public $users;
        }

    .. code-block:: xml

        <?xml version="1.0" encoding="UTF-8"?>
        <doctrine-mapping xmlns="http://doctrine-project.org/schemas/orm/doctrine-mapping"
              xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
              xsi:schemaLocation="http://doctrine-project.org/schemas/orm/doctrine-mapping
                                  http://www.doctrine-project.org/schemas/orm/doctrine-mapping.xsd">

            <entity name="Doctrine\Tests\Models\CMS\CmsGroup">
                <!-- ... -->
                <many-to-many field="users" target-entity="CmsUser" mapped-by="groups" fetch="EXTRA_LAZY" />
            </entity>
        </doctrine-mapping>

    .. code-block:: yaml

        Doctrine\Tests\Models\CMS\CmsGroup:
          type: entity
          # ...
          manyToMany:
            users:
              targetEntity: CmsUser
              mappedBy: groups
              fetch: EXTRA_LAZY
