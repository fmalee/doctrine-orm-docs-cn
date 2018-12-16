事件
======

Doctrine2具有一个属于 ``Common`` 包的一部分的轻量级事件系统。
Doctrine使用它来调度系统事件，主要是
:ref:`生命周期事件 <reference-events-lifecycle-events>`。
你也可以将它用于自己的自定义事件。

事件系统
----------------

事件系统由 ``EventManager`` 控制。它是Doctrine事件监听系统的核心。
监听器在此管理器上注册，并通过此管理器调度事件。

.. code-block:: php

    $evm = new EventManager();

现在我们可以添加一些事件监听器到 ``$evm``。让我们创建一个 ``TestEvent`` 类来玩玩。

.. code-block:: php

    class TestEvent
    {
        const preFoo = 'preFoo';
        const postFoo = 'postFoo';

        private $_evm;

        public $preFooInvoked = false;
        public $postFooInvoked = false;

        public function __construct($evm)
        {
            $evm->addEventListener(array(self::preFoo, self::postFoo), $this);
        }

        public function preFoo(EventArgs $e)
        {
            $this->preFooInvoked = true;
        }

        public function postFoo(EventArgs $e)
        {
            $this->postFooInvoked = true;
        }
    }

    // 创建一个实例
    $test = new TestEvent($evm);

可以使用 ``dispatchEvent()`` 方法来调度事件。

.. code-block:: php

    $evm->dispatchEvent(TestEvent::preFoo);
    $evm->dispatchEvent(TestEvent::postFoo);

你可以使用 ``removeEventListener()`` 方法来轻松删除监听器。

.. code-block:: php

    $evm->removeEventListener(array(self::preFoo, self::postFoo), $this);

Doctrine2事件系统也有一个简单的事件订阅器概念。
我们可以定义一个实现 ``\Doctrine\Common\EventSubscriber`` 接口的简单 ``TestEventSubscriber``
类，并实现一个返回已订阅的事件数组的 ``getSubscribedEvents()`` 方法。

.. code-block:: php

    class TestEventSubscriber implements \Doctrine\Common\EventSubscriber
    {
        public $preFooInvoked = false;

        public function preFoo()
        {
            $this->preFooInvoked = true;
        }

        public function getSubscribedEvents()
        {
            return array(TestEvent::preFoo);
        }
    }

    $eventSubscriber = new TestEventSubscriber();
    $evm->addEventSubscriber($eventSubscriber);

.. note::

    要在 ``getSubscribedEvents`` 方法中返回的数组是一个简单数组，其值为事件名称。
    订阅器必须具有与事件完全相同的命名方法。

现在，当你调度一个事件时，任何事件订阅器都将收到该事件的通知。

.. code-block:: php

    $evm->dispatchEvent(TestEvent::preFoo);

现在，你可以测试 ``$eventSubscriber`` 实例以查看是否已调用 ``preFoo()`` 方法。

.. code-block:: php

    if ($eventSubscriber->preFooInvoked) {
        echo 'pre foo invoked!';
    }

命名约定
~~~~~~~~~~~~~~~~~

与Doctrine2的 ``EventManager`` 一起使用的事件最好用小驼峰拼写法进行命名，相应常量的值应该是常量本身的名称，即使拼写也是如此。这有几个原因：

-  它很容易阅读
-  简洁
-  ``EventSubscriber`` 中的每个方法都以相应的常量值来命名。
   如果常量的名称和值不同，则与使用常量的意图相矛盾，使代码难以维护。

可以在上面的 ``TestEvent`` 例子中找到正确的表示法示例。

.. _reference-events-lifecycle-events:

生命周期事件
----------------

``EntityManager`` 和 ``UnitOfWork`` 在其已注册实体的生命周期内触发一系列事件。

-  ``preRemove`` - 在执行该实体的相应 ``EntityManager``
   删除操作之前，为给定实体发生 ``preRemove`` 事件。
   它不是为一个DQL ``DELETE`` 语句调用的。
-  ``postRemove`` - 删除一个实体后，该实体发生 ``postRemove`` 事件。
   它将在数据库删除操作后调用。它不是为DQL ``DELETE`` 语句调用的。
-  ``prePersist`` - 在执行一个给定实体的相应 ``EntityManager`` 持久操作之前，对于给定实体发生 ``prePersist`` 事件。
   应该注意，此事件仅在一个实体的 *初始* 持久时触发（即，它不会在将来的更新时触发）。
-  ``postPersist`` - 在一个实体持久化后，该实体发生 ``postPersist`` 事件。
   它将在数据库插入操作后调用。已生成的主键值在 ``postPersist`` 事件中可用。
-  ``preUpdate`` - ``preUpdate`` 事件发生在对实体数据的数据库 *更新* 操作 *之前*。
   它不是为一个DQL ``UPDATE`` 语句调用的，也不会在计算的更改集为空时调用。
-  ``postUpdate`` - ``postUpdate`` 事件发生在对实体数据的数据库 *更新* 操作 *之后*。
   它不是为一个DQL ``UPDATE`` 语句调用的。
-  ``postLoad`` - 在一个实体从数据库加载到当前 ``EntityManager``
   之后或者在对其应用刷新操作之后，该实体发生 ``postLoad`` 事件。
-  ``loadClassMetadata`` - 在从映射源（注释/xml/yaml）加载一个类的映射元数据之后发生
   ``loadClassMetadata`` 事件。此事件不是一个生命周期回调。
-  ``onClassMetadataNotFound`` - Loading class metadata for a particular
   requested class name failed.
   Manipulating the given event args instance
   allows providing fallback metadata even when no actual metadata exists
   or could be found.
   This event is not a lifecycle callback.
   加载一个特定请求的类名的类元数据失败。
   操作给定事件args实例允许提供回退元数据，即使没有实际元数据存在或可以找到。
   此事件不是生命周期回调。
-  ``preFlush`` - 该事件发生在一个刷新操作的最开始时。
-  ``onFlush`` - 在计算完所有已管理实体的更改集之后发生该事件。此事件不是一个生命周期回调。
-  ``postFlush`` - 该事件发生在一个刷新操作结束时。此事件不是一个生命周期回调。
-  ``onClear`` - 调用 ``EntityManager#clear()``
   操作时，在从工作单元中删除对实体的所有引用之后，发生 ``onClear`` 事件。此事件不是一个生命周期回调。

.. warning::

    请注意，使用 ``Doctrine\ORM\AbstractQuery#iterate()`` 时，``postLoad``
    事件将在对象被融合后立即执行，因此无法保证能初始化关联。
    组合使用 ``Doctrine\ORM\AbstractQuery#iterate()`` 和 ``postLoad`` 事件处理器是不安全的。

.. warning::

    请注意，如果你已将一个实体配置为级联删除关系，则 ``postRemove``
    事件或在删除一个实体后触发的任何事件都可以接收一个未初始化的代理。
    在这种情况下，你应该在已关联事件之前加载代理。

你可以从ORM包中的 ``Events`` 类中访问事件常量。

.. code-block:: php

    use Doctrine\ORM\Events;
    echo Events::preUpdate;

这些可以被 *两种* 不同类型的事件监听器挂钩起来：

-  生命周期回调是触发事件时调用的实体类的方法。从v2.4开始，他们会收到某种 ``EventArgs`` 实例。
-  生命周期事件监听器和订阅器是具有特定回调方法的类，它们会接收某种 ``EventArgs`` 实例。

监听器接收的 ``EventArgs`` 实例提供对实体、``EntityManager`` 和其他相关数据的访问。

.. note::

    在一个 ``EntityManager`` 的 ``flush()``
    期间发生的所有生命周期事件都对被许可的可执行操作具有非常具体的约束。请仔细阅读
    :ref:`reference-events-implementing-listeners` 章节，以了解哪些操作在哪个生命周期事件中被允许。

生命周期回调
-------------------

生命周期回调是在一个实体类上定义的。它们允许你在该实体类的一个实例遇到相关生命周期事件时触发回调。
可以为每个生命周期事件定义多个回调。生命周期回调最适用于一个特定实体类的生命周期的简单操作。

.. code-block:: php

    /** @Entity @HasLifecycleCallbacks */
    class User
    {
        // ...

        /**
         * @Column(type="string", length=255)
         */
        public $value;

        /** @Column(name="created_at", type="string", length=255) */
        private $createdAt;

        /** @PrePersist */
        public function doStuffOnPrePersist()
        {
            $this->createdAt = date('Y-m-d H:i:s');
        }

        /** @PrePersist */
        public function doOtherStuffOnPrePersist()
        {
            $this->value = 'changed from prePersist callback!';
        }

        /** @PostPersist */
        public function doStuffOnPostPersist()
        {
            $this->value = 'changed from postPersist callback!';
        }

        /** @PostLoad */
        public function doStuffOnPostLoad()
        {
            $this->value = 'changed from postLoad callback!';
        }

        /** @PreUpdate */
        public function doStuffOnPreUpdate()
        {
            $this->value = 'changed from preUpdate callback!';
        }
    }

请注意，设置为生命周期回调的方法需要是 ``public``，并且在使用这些注释时，你必须在实体类上标记
``@HasLifecycleCallbacks`` 注释。

如果要从YAML或XML注册生命周期回调，可以使用以下命令执行此操作。

.. code-block:: yaml

    User:
      type: entity
      fields:
    # ...
        name:
          type: string(50)
      lifecycleCallbacks:
        prePersist: [ doStuffOnPrePersist, doOtherStuffOnPrePersist ]
        postPersist: [ doStuffOnPostPersist ]

在YAML中，``lifecycleCallbacks`` 项的 ``key``
是你要触发的事件，而该值是要调用的方法（可以使多个方法）。
允许的事件类型是之前的“生命周期事件”部分中列出的事件类型。

XML看起来像这样：

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>

    <doctrine-mapping xmlns="http://doctrine-project.org/schemas/orm/doctrine-mapping"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://doctrine-project.org/schemas/orm/doctrine-mapping
                              /Users/robo/dev/php/Doctrine/doctrine-mapping.xsd">

        <entity name="User">

            <lifecycle-callbacks>
                <lifecycle-callback type="prePersist" method="doStuffOnPrePersist"/>
                <lifecycle-callback type="postPersist" method="doStuffOnPostPersist"/>
            </lifecycle-callbacks>

        </entity>

    </doctrine-mapping>

在XML中，``lifecycle-callback`` 项的 ``type`` 是你要触发的事件，而 ``method`` 是要调用的方法。
允许的事件类型是之前的“生命周期事件”部分中列出的事件类型。

使用YAML或XML时，你需要记得创建 ``public`` 方法以匹配你定义的回调名称。
例如，在这些例子中，你需要在 ``User`` 模型中定义
``doStuffOnPrePersist()``、``doOtherStuffOnPrePersist()``
以及 ``doStuffOnPostPersist()`` 方法。

.. code-block:: php

    // ...

    class User
    {
        // ...

        public function doStuffOnPrePersist()
        {
            // ...
        }

        public function doOtherStuffOnPrePersist()
        {
            // ...
        }

        public function doStuffOnPostPersist()
        {
            // ...
        }
    }


生命周期回调事件参数
-----------------------------------

.. versionadded:: 2.4

自2.4开始，使用 ``lifecycle-callback`` 来触发事件。

使用附加参数，你可以在这些回调方法中访问 ``EntityManager`` 和 ``UnitOfWorkAPI`` API。

.. code-block:: php

    // ...

    class User
    {
        public function preUpdate(PreUpdateEventArgs $event)
        {
            if ($event->hasChangedField('username')) {
                // 用户名已更改时执行某些操作。
            }
        }
    }

监听和订阅生命周期事件
---------------------------------------------

生命周期事件监听器比在实体类上定义的简单生命周期回调更强大。
它们位于实体之上的某个级别，允许你跨不同的实体类实现可重用的行为。

请注意，它们需要有关 ``EntityManager`` 和 ``UnitOfWork`` 内部运作的更详细的知识。
如果你正在尝试编写自己的监听器，请仔细阅读 :ref:`reference-events-implementing-listeners` 部分。

对于事件订阅器，没有任何惊喜。
它们在自身的 ``getSubscribedEvents`` 方法中声明生命周期事件，并提供拥有期望的相关参数的公共方法。

一个生命周期事件监听器如下所示：

.. code-block:: php

    use Doctrine\Common\Persistence\Event\LifecycleEventArgs;

    class MyEventListener
    {
        public function preUpdate(LifecycleEventArgs $args)
        {
            $entity = $args->getObject();
            $entityManager = $args->getObjectManager();

            // 也许你只想对一些“Product”实体采取行动
            if ($entity instanceof Product) {
                // 使用 Product 做一些处理
            }
        }
    }

一个生命周期事件订阅器可能如下所示：

.. code-block:: php

    use Doctrine\ORM\Events;
    use Doctrine\Common\EventSubscriber;
    use Doctrine\Common\Persistence\Event\LifecycleEventArgs;

    class MyEventSubscriber implements EventSubscriber
    {
        public function getSubscribedEvents()
        {
            return array(
                Events::postUpdate,
            );
        }

        public function postUpdate(LifecycleEventArgs $args)
        {
            $entity = $args->getObject();
            $entityManager = $args->getObjectManager();

            // 也许你只想对一些“Product”实体采取行动
            if ($entity instanceof Product) {
                // 使用 Product 做一些处理
            }
        }
    }

.. note::

    所有的实体都会触发生命周期事件。监听器和订阅器有责任检查实体是否属于它想要处理的类型。

要注册事件监听器或订阅器，你必须将其挂钩到传递给 ``EntityManager`` 工厂的 ``EventManager``：

.. code-block:: php

    $eventManager = new EventManager();
    $eventManager->addEventListener(array(Events::preUpdate), new MyEventListener());
    $eventManager->addEventSubscriber(new MyEventSubscriber());

    $entityManager = EntityManager::create($dbOpts, $config, $eventManager);

你还可以在创建 ``EntityManager`` 后检索事件管理器实例：

.. code-block:: php

    $entityManager->getEventManager()->addEventListener(array(Events::preUpdate), new MyEventListener());
    $entityManager->getEventManager()->addEventSubscriber(new MyEventSubscriber());

.. _reference-events-implementing-listeners:

实现事件监听器
----------------------------

本节将介绍 ``UnitOfWork`` 的生命周期事件，以及在特定生命周期事件期间的限制。
虽然你在所有这些事件中都传递了 ``EntityManager``，但你必须非常小心地遵循这些限制，因为在错误事件中的操作可能会产生许多不同的错误，例如数据不一致以及 *丢失* 更新/持久/删除。

这些限制也适用于所描述的生命周期回调事件，并且附加了限制（在版本2.4之前）：即你无法访问这些事件中的 ``EntityManager`` 或 ``UnitOfWork`` API。
For the described events that are also lifecycle callback events
the restrictions apply as well, with the additional restriction
that (prior to version 2.4) you do not have access to the
EntityManager or UnitOfWork APIs inside these events.

prePersist
~~~~~~~~~~

有两种方式可以触发 ``prePersist`` 事件。最明显的是当你调用 ``EntityManager#persist()``
时。而所有级联关联也会调用该事件。

当计算关联的更改并将此关联标记为级联持久时，在 ``flush()`` 方法内部还有另一种方法可调用 ``prePersist``。
在此操作期间找到的任何新实体也会被持久化并prePersist调用它。这称为“可达性持久性”。
There is another way for ``prePersist`` to be called, inside the
``flush()`` method when changes to associations are computed and
this association is marked as cascade persist. Any new entity found
during this operation is also persisted and ``prePersist`` called
on it. This is called "persistence by reachability".

在这两种情况下，你都会被传递一个可以访问实体和实体管理器的 ``LifecycleEventArgs`` 实例。

以下限制适用于 ``prePersist``：

-  如果你使用一个 **PrePersist标识生成器**（如序列），则该ID值将 *不会* 在任何 ``prePersist`` 事件中可用。
-  Doctrine不会承认在一个 ``prePersist`` 事件中对关系的改变。这包括对一个集合的修改，例如添加、删除或替换。

preRemove
~~~~~~~~~

在将每个实体传递给 ``EntityManager#remove()`` 方法时，都会调用 ``preRemove`` 事件。
它被级联为标记为级联删除的所有关联。
The ``preRemove`` event is called on every entity when its passed
to the ``EntityManager#remove()`` method.
It is cascaded for all associations that are marked as cascade delete.

除了在一个刷新操作期间调用 ``remove`` 方法本身时，对 ``preRemove`` 事件内部可以调用的方法没有限制。

preFlush
~~~~~~~~

preFlushEntityManager#flush()在其他任何事情之前被召唤。可以在其监听器内安全地调用 ``EntityManager#flush()``。
``preFlush`` is called at ``EntityManager#flush()`` before
anything else. ``EntityManager#flush()`` can be called safely
inside its listeners.

.. code-block:: php

    use Doctrine\ORM\Event\PreFlushEventArgs;

    class PreFlushExampleListener
    {
        public function preFlush(PreFlushEventArgs $args)
        {
            // ...
        }
    }

onFlush
~~~~~~~

``OnFlush`` 是一个非常强大的事件。在对所有管理实体进行更改并计算其关联后，将在 ``EntityManager#flush()`` 内部调用它。这意味着，该 事件可以访问以下集合：
OnFlush is a very powerful event. It is called inside
``EntityManager#flush()`` after the changes to all the managed
entities and their associations have been computed. This means, the
``onFlush`` event has access to the sets of:

-  计划插入的实体
-  计划更新的实体
-  计划删除的实体
-  计划更新的集合
-  计划删除的集合

要使用 ``onFlush`` 事件，你必须熟悉内部 ``UnitOfWork`` API，以便授予你访问前面提到的集合的权限。看这个例子：
To make use of the onFlush event you have to be familiar with the
internal UnitOfWork API, which grants you access to the previously
mentioned sets. See this example:

.. code-block:: php

    class FlushExampleListener
    {
        public function onFlush(OnFlushEventArgs $eventArgs)
        {
            $em = $eventArgs->getEntityManager();
            $uow = $em->getUnitOfWork();

            foreach ($uow->getScheduledEntityInsertions() as $entity) {

            }

            foreach ($uow->getScheduledEntityUpdates() as $entity) {

            }

            foreach ($uow->getScheduledEntityDeletions() as $entity) {

            }

            foreach ($uow->getScheduledCollectionDeletions() as $col) {

            }

            foreach ($uow->getScheduledCollectionUpdates() as $col) {

            }
        }
    }

以下限制适用于 ``onFlush`` 事件：

-  If you create and persist a new entity in ``onFlush``, then
   calling ``EntityManager#persist()`` is not enough.
   You have to execute an additional call to
   ``$unitOfWork->computeChangeSet($classMetadata, $entity)``.
   如果你在 ``onFlush`` 中创建并持久一个新实体，则调用 ``EntityManager#persist()``
   是不够的。你必须执行一个额外的调用到 ``$unitOfWork->computeChangeSet($classMetadata, $entity)``。
-  Changing primitive fields or associations requires you to
   explicitly trigger a re-computation of the changeset of the
   affected entity. This can be done by calling
   ``$unitOfWork->recomputeSingleEntityChangeSet($classMetadata, $entity)``.
   更改原始字段或关联，要求你显式触发受影响实体的变更集的重新计算。这可以通过调用
   ``$unitOfWork->recomputeSingleEntityChangeSet($classMetadata, $entity)`` 来完成。

postFlush
~~~~~~~~~

在 ``EntityManager#flush()`` 结束时会调用 ``postFlush``。
*不能* 在它的监听器内部安全地调用 ``EntityManager#flush()``。

.. code-block:: php

    use Doctrine\ORM\Event\PostFlushEventArgs;

    class PostFlushExampleListener
    {
        public function postFlush(PostFlushEventArgs $args)
        {
            // ...
        }
    }

preUpdate
~~~~~~~~~

``PreUpdate`` 是使用事件最严格的事件，因为它是在为 ``EntityManager#flush()`` 方法内的实体调用 ``update`` 语句之前调用的。
请注意，当计算的变更集为空时，不会触发此事件。
PreUpdate is the most restrictive to use event, since it is called
right before an update statement is called for an entity inside the
``EntityManager#flush()`` method. Note that this event is not
triggered when the computed changeset is empty.

在此事件中永远不允许对更新实体的关联进行更改，因为在刷新操作的此时，Doctrine无法保证正确处理引用完整性。
此事件具有强大的功能，但它使用 ``PreUpdateEventArgs`` 实例执行，该实例包含对此实体的计算更改集的引用。
Changes to associations of the updated entity are never allowed in
this event, since Doctrine cannot guarantee to correctly handle
referential integrity at this point of the flush operation. This
event has a powerful feature however, it is executed with a
``PreUpdateEventArgs`` instance, which contains a reference to the
computed change-set of this entity.

这意味着你可以使用旧值和新值访问已为此实体更改的所有字段。以下方法可用于 ``PreUpdateEventArgs``：
This means you have access to all the fields that have changed for
this entity with their old and new value. The following methods are
available on the ``PreUpdateEventArgs``:

-  ``getEntity()`` 获得对实际实体的访问权限。
-  ``getEntityChangeSet()`` 获取变更集数组的一个副本。对此返回数组的更改不会影响更新。
-  ``hasChangedField($fieldName)`` 检查当前实体的给定字段名称是否已更改。
-  ``getOldValue($fieldName)`` 和 ``getNewValue($fieldName)`` 访问一个字段的值。
-  ``setNewValue($fieldName, $value)`` 更改一个要将更新的字段的值。

此事件的一个简单示例如下所示：

.. code-block:: php

    class NeverAliceOnlyBobListener
    {
        public function preUpdate(PreUpdateEventArgs $eventArgs)
        {
            if ($eventArgs->getEntity() instanceof User) {
                if ($eventArgs->hasChangedField('name') && $eventArgs->getNewValue('name') == 'Alice') {
                    $eventArgs->setNewValue('name', 'Bob');
                }
            }
        }
    }

你还可以使用此监听器来实现已更改的所有字段的验证。
当有昂贵(expensive)的验证要调用时，这比使用生命周期回调更有效：

.. code-block:: php

    class ValidCreditCardListener
    {
        public function preUpdate(PreUpdateEventArgs $eventArgs)
        {
            if ($eventArgs->getEntity() instanceof Account) {
                if ($eventArgs->hasChangedField('creditCard')) {
                    $this->validateCreditCard($eventArgs->getNewValue('creditCard'));
                }
            }
        }

        private function validateCreditCard($no)
        {
            // 抛出异一个常来中断flush事件。事件将被回滚。
        }
    }

此事件的限制：

-  Changes to associations of the passed entities are not
   recognized by the flush operation anymore.
   刷新操作不再识别对已传递实体的关联的更改。
-  Changes to fields of the passed entities are not recognized by
   the flush operation anymore, use the computed change-set passed to
   the event to modify primitive field values, e.g. use
   ``$eventArgs->setNewValue($field, $value);`` as in the Alice to Bob example above.
   刷新操作不再识别对传递的实体的字段的改变，使用传递给事件的计算的改变集来修改原始字段值，例如 在上面的Alice to Bob示例中使用 ``$eventArgs->setNewValue($field, $value);``。
-  Any calls to ``EntityManager#persist()`` or
   ``EntityManager#remove()``, even in combination with the UnitOfWork
   API are strongly discouraged and don't work as expected outside the
   flush operation.
   强烈建议不要调用EntityManager#persist()或 EntityManager#remove()甚至与UnitOfWork API结合使用，并且在刷新操作之外不能按预期工作。

postUpdate, postRemove, postPersist
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

``EntityManager#flush()`` 内部调用了三个后置事件。
此处的更改与数据库中的持久性无关，但你可以使用这些事件来更改非可持久化的项，例如非映射字段、日志记录甚至是未由Doctrine直接映射的关联类。

postLoad
~~~~~~~~

在 ``EntityManager`` 构造一个实体之后调用此事件。

实体监听器
----------------

.. versionadded:: 2.4

一个实体监听器是用于一个实体的生命周期监听器类。

- 实体监听器的映射可以应用于实体类或已映射超类。
- An entity listener is defined by mapping the entity class with the corresponding mapping.通过将实体类映射到相应的映射来定义一个实体监听器。

.. configuration-block::

    .. code-block:: php

        <?php
        namespace MyProject\Entity;

        /** @Entity @EntityListeners({"UserListener"}) */
        class User
        {
            // ....
        }
    .. code-block:: xml

        <doctrine-mapping>
            <entity name="MyProject\Entity\User">
                <entity-listeners>
                    <entity-listener class="UserListener"/>
                </entity-listeners>
                <!-- .... -->
            </entity>
        </doctrine-mapping>
    .. code-block:: yaml

        MyProject\Entity\User:
          type: entity
          entityListeners:
            UserListener:
          # ....

.. _reference-entity-listeners:

实体监听器类
~~~~~~~~~~~~~~~~~~~~~~

一个 ``Entity Listener`` 可以是任何类，默认情况下它应该是一个带有无参数构造函数的类。
An ``Entity Listener`` could be any class, by default it should be a class with a no-arg constructor.

- Different from :ref:`reference-events-implementing-listeners` an ``Entity Listener`` is invoked just to the specified entity.从不同的实现事件监听器的调用只是指定的实体Entity Listener
- An entity listener method receives two arguments, the entity instance and the lifecycle event.实体监听器方法接收两个参数，即实体实例和生命周期事件。
- The callback method can be defined by naming convention or specifying a method mapping.可以通过命名约定或指定方法映射来定义回调方法。
- When a listener mapping is not given the parser will use the naming convention to look for a matching method,
  e.g. it will look for a public ``preUpdate()`` method if you are listening to the ``preUpdate`` event.
  当没有给出监听器映射时，解析器将使用命名约定来查找匹配方法，例如，preUpdate()如果你正在侦听preUpdate事件，它将查找公共方法。
- When a listener mapping is given the parser will not look for any methods using the naming convention.当给出监听器映射时，解析器将不会使用命名约定查找任何方法。

.. code-block:: php

    class UserListener
    {
        public function preUpdate(User $user, PreUpdateEventArgs $event)
        {
            // Do something on pre update.
        }
    }

要定义特定的事件监听器方法（不遵循命名约定的方法），你需要使用事件类型映射映射监听器方法：
To define a specific event listener method (one that does not follow the naming convention)
you need to map the listener method using the event type mapping:

.. configuration-block::

    .. code-block:: php

        class UserListener
        {
            /** @PrePersist */
            public function prePersistHandler(User $user, LifecycleEventArgs $event) { // ... }

            /** @PostPersist */
            public function postPersistHandler(User $user, LifecycleEventArgs $event) { // ... }

            /** @PreUpdate */
            public function preUpdateHandler(User $user, PreUpdateEventArgs $event) { // ... }

            /** @PostUpdate */
            public function postUpdateHandler(User $user, LifecycleEventArgs $event) { // ... }

            /** @PostRemove */
            public function postRemoveHandler(User $user, LifecycleEventArgs $event) { // ... }

            /** @PreRemove */
            public function preRemoveHandler(User $user, LifecycleEventArgs $event) { // ... }

            /** @PreFlush */
            public function preFlushHandler(User $user, PreFlushEventArgs $event) { // ... }

            /** @PostLoad */
            public function postLoadHandler(User $user, LifecycleEventArgs $event) { // ... }
        }

    .. code-block:: xml

        <doctrine-mapping>
            <entity name="MyProject\Entity\User">
                 <entity-listeners>
                    <entity-listener class="UserListener">
                        <lifecycle-callback type="preFlush"      method="preFlushHandler"/>
                        <lifecycle-callback type="postLoad"      method="postLoadHandler"/>

                        <lifecycle-callback type="postPersist"   method="postPersistHandler"/>
                        <lifecycle-callback type="prePersist"    method="prePersistHandler"/>

                        <lifecycle-callback type="postUpdate"    method="postUpdateHandler"/>
                        <lifecycle-callback type="preUpdate"     method="preUpdateHandler"/>

                        <lifecycle-callback type="postRemove"    method="postRemoveHandler"/>
                        <lifecycle-callback type="preRemove"     method="preRemoveHandler"/>
                    </entity-listener>
                </entity-listeners>
                <!-- .... -->
            </entity>
        </doctrine-mapping>

    .. code-block:: yaml

        MyProject\Entity\User:
          type: entity
          entityListeners:
            UserListener:
              preFlush: [preFlushHandler]
              postLoad: [postLoadHandler]

              postPersist: [postPersistHandler]
              prePersist: [prePersistHandler]

              postUpdate: [postUpdateHandler]
              preUpdate: [preUpdateHandler]

              postRemove: [postRemoveHandler]
              preRemove: [preRemoveHandler]
          # ....

.. note::

    不保证同一事件（例如多个 ``@PrePersist``）的多个方法的执行顺序。

实体监听器解析器
~~~~~~~~~~~~~~~~~~~~~~~~~~

Doctrine调用监听器解析器来获取监听器实例。

- 一个解析器允许你注册一个特定的实体监听器实例。
- 你还可以通过扩展 ``Doctrine\ORM\Mapping\DefaultEntityListenerResolver`` 或实现
  ``Doctrine\ORM\Mapping\EntityListenerResolver`` 来实现自己的解析器。

指定一个实体监听器实例：

.. code-block:: php

    <?php
    // User.php

    /** @Entity @EntityListeners({"UserListener"}) */
    class User
    {
        // ....
    }

    // UserListener.php
    class UserListener
    {
        public function __construct(MyService $service)
        {
            $this->service = $service;
        }

        public function preUpdate(User $user, PreUpdateEventArgs $event)
        {
            $this->service->doSomething($user);
        }
    }

    // register a entity listener.
    $listener = $container->get('user_listener');
    $em->getConfiguration()->getEntityListenerResolver()->register($listener);

实现自己的解析器：

.. code-block:: php

    <?php
    class MyEntityListenerResolver extends \Doctrine\ORM\Mapping\DefaultEntityListenerResolver
    {
        public function __construct($container)
        {
            $this->container = $container;
        }

        public function resolve($className)
        {
            // resolve the service id by the given class name;
            $id = 'user_listener';

            return $this->container->get($id);
        }
    }

    // Configure the listener resolver only before instantiating the EntityManager
    $configurations->setEntityListenerResolver(new MyEntityListenerResolver);
    EntityManager::create(.., $configurations, ..);

加载 ``ClassMetadata`` 事件
------------------------------

读取一个实体的映射信息时，会将其填充到 ``ClassMetadataInfo`` 实例中。你可以挂钩到此过程并操纵该实例。
When the mapping information for an entity is read, it is populated
in to a ``ClassMetadataInfo`` instance. You can hook in to this
process and manipulate the instance.

.. code-block:: php

    $test = new TestEvent();
    $metadataFactory = $em->getMetadataFactory();
    $evm = $em->getEventManager();
    $evm->addEventListener(Events::loadClassMetadata, $test);

    class TestEvent
    {
        public function loadClassMetadata(\Doctrine\ORM\Event\LoadClassMetadataEventArgs $eventArgs)
        {
            $classMetadata = $eventArgs->getClassMetadata();
            $fieldMapping = array(
                'fieldName' => 'about',
                'type' => 'string',
                'length' => 255
            );
            $classMetadata->mapField($fieldMapping);
        }
    }
