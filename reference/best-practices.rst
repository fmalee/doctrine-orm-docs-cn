最佳实践
==============

这里提到的影响数据库设计的最佳实践通常是指使用Doctrine时的最佳实践，并不一定反映数据库设计的最佳实践。

尽可能地约束关系
-------------------------------------------

尽可能地约束关系很重要。这意味着：

-  施加一个遍历方向（如果可能，避免双向关联）
-  消除不必要的关联

这有几个好处：

-  减少域模型中的耦合
-  域模型中的代码更简单（无需正确维护双向性）
-  减少对Doctrine的工作

避免复合键
--------------------

尽管Doctrine完全支持复合键，但最好尽可能的不使用它们。
复合键需要Doctrine做额外工作，因此错误率更高。

谨慎地使用事件
----------------------

Doctrine的事件系统是强大而快速的。如果大量使用事件（尤其是生命周期事件），会对应用的性能产生负面影响。
因此，你应该明智地使用事件。

谨慎地使用级联
------------------------

自动级联的持久、删除、合并等操作非常方便，但应谨慎地使用。
*不要* 简单地将所有级联添加到所有关联中。要考虑最有可能使用的场景以及哪些级联实际上对特定关联有意义。

不要使用特殊字符
----------------------------

避免在类、字段、表或列名称中使用任何非ASCII字符。
Doctrine本身在很多地方都不是unicode-safe，直到PHP本身完全具有unicode-aware。

不要使用标识符引用
----------------------------

标识符引用是使用保留字的变通方法，这通常会导致边缘情况出现问题。
不要使用标识符引用，并避免使用保留字作为表或列名。

在构造函数中初始化集合
-----------------------------------------

推荐的最佳实践是在实体的构造函数中初始化任何业务集合。例如：

.. code-block:: php

    namespace MyProject\Model;
    use Doctrine\Common\Collections\ArrayCollection;

    class User {
        private $addresses;
        private $articles;

        public function __construct() {
            $this->addresses = new ArrayCollection;
            $this->articles = new ArrayCollection;
        }
    }

不要将外键映射到实体的字段中
---------------------------------------------

外键在一个对象模型中没有任何意义。外键是关系数据库建立关系的方式，而你的对象模型通过对象引用来建立关系。
因此，将外键映射到对象字段会严重地将关系模型的细节泄漏到对象模型中。这是你真正不应该做的事情。

使用显式的事务界限
------------------------------------

虽然Doctrine会自动将所有DML操作封装在 ``flush()`` 的事务中，但最好自己显式的设置事务边界。
否则，每个查询都封装在一个小事务中（是的，即使是 ``SELECT`` 查询），因为你无法在事务之外与数据库通信。
虽然这种用于只读（``SELECT``）查询的短事务通常没有任何明显的性能影响，但仍然推荐使用通过显式事务边界建立的较少的、定义良好的事务。
While Doctrine will automatically wrap all DML operations in a
transaction on flush(), it is considered best practice to
explicitly set the transaction boundaries yourself.
Otherwise every single query is wrapped in a small transaction (Yes, SELECT
queries, too) since you can not talk to your database outside of a
transaction. While such short transactions for read-only (SELECT)
queries generally don't have any noticeable performance impact, it
is still preferable to use fewer, well-defined transactions that
are established through explicit transaction boundaries.
