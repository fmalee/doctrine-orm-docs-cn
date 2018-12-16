变更跟踪策略
========================

变更跟踪是确定自上次与数据库同步以来已管理实体中发生了哪些变化的过程。

Doctrine提供了 *3* 种不同的变更跟踪策略，每种策略都有其独特的优点和缺点。
可以基于每个类（或更准确地说，每个层级）定义变更跟踪策略。

Deferred Implicit
延迟隐含
~~~~~~~~~~~~~~~~~

延迟隐式策略是默认的变更跟踪策略，也是最方便的策略。
使用此策略，Doctrine会在提交时通过逐个比较属性来检测更改，并检测对其他已管理实体引用的实体或新实体的更改（“可达性持久”）。
虽然这是最方便的策略，但如果你要处理大型工作单元（请参阅“了解工作单元”），它可能会对性能产生负面影响。
由于Doctrine无法知道更改了什么，因此每次调用 ``EntityManager#flush()``
时都需要检查所有已管理实体的更改，这使得此操作成本相当高。

Deferred Explicit
延迟显式
~~~~~~~~~~~~~~~~~

延迟显式策略类似于延迟隐式策略，因为它也是在提交时通过逐个比较属性来检测更改。
不同之处在于，Doctrine2仅考虑那些通过调用 ``EntityManager#persist(entity)``
或通过一个保存级联来明确标记为更改的实体。该策略将跳过所有其他实体。
因此，该策略为大型工作单元提供了性能改进，同时牺牲了“自动脏检查”行为。

因此，此策略可能会使 ``flush()`` 操作更划算。
这样做的负面影响是，如果你有一个相当大型的应用，并且为了处理业务任务而将你的对象传递到多个层，
则可能需要自己跟踪哪些实体发生了变化，以便将它们传递给 ``EntityManager#persist()``。

可以这样来配置此策略：

.. code-block:: php

    /**
     * @Entity
     * @ChangeTrackingPolicy("DEFERRED_EXPLICIT")
     */
    class User
    {
        // ...
    }

通知
~~~~~~

该政策基于这样的假设：实体通知感兴趣的监听器其属性的变化。
为此，想要使用此策略的类需要从Doctrine命名空间实现 ``NotifyPropertyChanged`` 接口。
作为指导原则，这样的实现可以如下所示：
This policy is based on the assumption that the entities notify
interested listeners of changes to their properties. For that
purpose, a class that wants to use this policy needs to implement
the ``NotifyPropertyChanged`` interface from the Doctrine
namespace. As a guideline, such an implementation can look as
follows:

.. code-block:: php

    use Doctrine\Common\NotifyPropertyChanged,
        Doctrine\Common\PropertyChangedListener;

    /**
     * @Entity
     * @ChangeTrackingPolicy("NOTIFY")
     */
    class MyEntity implements NotifyPropertyChanged
    {
        // ...

        private $_listeners = array();

        public function addPropertyChangedListener(PropertyChangedListener $listener)
        {
            $this->_listeners[] = $listener;
        }
    }

然后，在此类或派生类的每个属性setter中，你需要通知所有的 ``PropertyChangedListener`` 实例。
作为一个例子，我们在 ``MyEntity`` 中添加了一个方便的方法来展示这种行为：

.. code-block:: php

    // ...

    class MyEntity implements NotifyPropertyChanged
    {
        // ...

        protected function _onPropertyChanged($propName, $oldValue, $newValue)
        {
            if ($this->_listeners) {
                foreach ($this->_listeners as $listener) {
                    $listener->propertyChanged($this, $propName, $oldValue, $newValue);
                }
            }
        }

        public function setData($data)
        {
            if ($data != $this->data) {
                $this->_onPropertyChanged('data', $this->data, $data);
                $this->data = $data;
            }
        }
    }

你必须在 ``MyEntity`` 的每个改变持久状态的方法中调用 ``_onPropertyChanged``。

检查新值是否与旧值不同并非强制要求，但建议使用。这样，在考虑一个更改属性是你可以有完全的控制权。

这个策略的负面意义是显而易见的：你需要实现一个接口并编写一些管道代码。
但请注意，我们努力保持此通知功能的抽象。
严格来说，它与持久层和Doctrine ORM或DBAL无关。你可能会发现属性通知事件在许多其他情况下也会派上用场。
如前所述，``Doctrine\Common`` 命名空间并不是那么邪恶，只包含几乎没有外部依赖关系的非常小的类和接口（没有DBAL，没有ORM），如果你想换掉，你可以很容易地接受它持久层。
此变更跟踪策略不会引入对Doctrine DBAL/ORM或持久层的依赖。
The negative point of this policy is obvious: You need implement an
interface and write some plumbing code. But also note that we tried
hard to keep this notification functionality abstract. Strictly
speaking, it has nothing to do with the persistence layer and the
Doctrine ORM or DBAL. You may find that property notification
events come in handy in many other scenarios as well. As mentioned
earlier, the ``Doctrine\Common`` namespace is not that evil and
consists solely of very small classes and interfaces that have
almost no external dependencies (none to the DBAL and none to the
ORM) and that you can easily take with you should you want to swap
out the persistence layer. This change tracking policy does not
introduce a dependency on the Doctrine DBAL/ORM or the persistence
layer.

该政策的积极点和主要优势在于其有效性。
它具有3个策略的最佳性能特征，具有更大的工作单元，并且当没有任何改变时，``flush()`` 操作非常划算。
The positive point and main advantage of this policy is its
effectiveness. It has the best performance characteristics of the 3
policies with larger units of work and a flush() operation is very
cheap when nothing has changed.
