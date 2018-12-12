复合和外键作为主键
=========================================

.. versionadded:: 2.1

Doctrine2原生支持复合主键。复合键是一个非常强大的关系数据库概念，我们非常注意确保Doctrine2支持尽可能多的复合主键用例。
对于Doctrine2.0，支持原始数据类型的复合键，Doctrine2.1甚至支持外键作为主键。

本教程将介绍复合主键的语义如何工作以及它们如何映射到数据库。

一般事项
~~~~~~~~~~~~~~~~~~~~~~

具有复合键的每个实体都不能使用除 ``NONE`` 之外的id生成器。
这意味着必须在调用 ``EntityManager#persist($entity)`` 之前设置该ID字段的值。

仅限原始类型
~~~~~~~~~~~~~~~~~~~~

即使在2.0版本中，只要它们只包含基本类型 ``integer`` 和 ``string``，你就可以拥有复合键。
假设你要创建汽车数据库并使用型号名称和生产年份作为主键：

.. configuration-block::

    .. code-block:: php

        <?php
        namespace VehicleCatalogue\Model;

        /**
         * @Entity
         */
        class Car
        {
            /** @Id @Column(type="string") */
            private $name;
            /** @Id @Column(type="integer") */
            private $year;

            public function __construct($name, $year)
            {
                $this->name = $name;
                $this->year = $year;
            }

            public function getModelName()
            {
                return $this->name;
            }

            public function getYearOfProduction()
            {
                return $this->year;
            }
        }

    .. code-block:: xml

        <?xml version="1.0" encoding="UTF-8"?>
        <doctrine-mapping xmlns="http://doctrine-project.org/schemas/orm/doctrine-mapping"
              xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
              xsi:schemaLocation="http://doctrine-project.org/schemas/orm/doctrine-mapping
                                  http://www.doctrine-project.org/schemas/orm/doctrine-mapping.xsd">

            <entity name="VehicleCatalogue\Model\Car">
                <id field="name" type="string" />
                <id field="year" type="integer" />
            </entity>
        </doctrine-mapping>

    .. code-block:: yaml

        VehicleCatalogue\Model\Car:
          type: entity
          id:
            name:
              type: string
            year:
              type: integer

现在你可以使用此实体：

.. code-block:: php

    namespace VehicleCatalogue\Model;

    // $em 既是 EntityManager

    $car = new Car("Audi A8", 2010);
    $em->persist($car);
    $em->flush();

对于查询，你可以将数组用于 ``DQL`` 和 ``EntityRepositories``：

.. code-block:: php

    namespace VehicleCatalogue\Model;

    // $em 既是 EntityManager
    $audi = $em->find("VehicleCatalogue\Model\Car", array("name" => "Audi A8", "year" => 2010));

    $dql = "SELECT c FROM VehicleCatalogue\Model\Car c WHERE c.id = ?1";
    $audi = $em->createQuery($dql)
               ->setParameter(1, array("name" => "Audi A8", "year" => 2010))
               ->getSingleResult();

你还可以在关联中使用此实体。然后Doctrine会生成两个外键：``name`` 和 相关实体的 ``year``。

.. note::

    此示例展示了如何在 ``EntityManager#persist()`` 之前很好地解决现有值的要求：将它们添加为构造函数的必需值。

通过外部实体进行标识
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. note::

    只有Doctrine 2.1支持通过外部实体进行进行标识

这里有大量的用例，其中一个实体的标识应由一个或多个父实体的实体确定。

-   一个实体的动态属性（例如 ``Article``）。
    每个文章都有许多属性，主键为 ``article_id`` 和 ``attribute_name``。
-   一个 ``Person`` 的 ``Address`` 对象，``address`` 的主键是 ``user_id``。
    这不是复合主键的用例，但该标识是通过一个外部实体和一个外键派生的。
-   具有元数据的连接表可以被建模为实体，例如具有少量描述和评分的两篇文章之间的连接。

通过外部实体映射identity的语义很简单：

-   仅允许 *多对一* 或 *一对一* 关联。
-   将 ``@Id`` 注释插入每个关联。
-   使用XML中的关联字段名称设置一个 ``association-key`` 属性。
-   在YAML中使用关联的字段名称设置一个 ``associationKey:`` 键。

用例1：动态属性
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

我们看看带有任意属性的文章的示例，该映射如下所示：

.. configuration-block::

    .. code-block:: php

        <?php
        namespace Application\Model;

        use Doctrine\Common\Collections\ArrayCollection;

        /**
         * @Entity
         */
        class Article
        {
            /** @Id @Column(type="integer") @GeneratedValue */
            private $id;
            /** @Column(type="string") */
            private $title;

            /**
             * @OneToMany(targetEntity="ArticleAttribute", mappedBy="article", cascade={"ALL"}, indexBy="attribute")
             */
            private $attributes;

            public function addAttribute($name, $value)
            {
                $this->attributes[$name] = new ArticleAttribute($name, $value, $this);
            }
        }

        /**
         * @Entity
         */
        class ArticleAttribute
        {
            /** @Id @ManyToOne(targetEntity="Article", inversedBy="attributes") */
            private $article;

            /** @Id @Column(type="string") */
            private $attribute;

            /** @Column(type="string") */
            private $value;

            public function __construct($name, $value, $article)
            {
                $this->attribute = $name;
                $this->value = $value;
                $this->article = $article;
            }
        }

    .. code-block:: xml

        <doctrine-mapping xmlns="http://doctrine-project.org/schemas/orm/doctrine-mapping"
              xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
              xsi:schemaLocation="http://doctrine-project.org/schemas/orm/doctrine-mapping
                            http://doctrine-project.org/schemas/orm/doctrine-mapping.xsd">

             <entity name="Application\Model\ArticleAttribute">
                <id name="article" association-key="true" />
                <id name="attribute" type="string" />

                <field name="value" type="string" />

                <many-to-one field="article" target-entity="Article" inversed-by="attributes" />
             <entity>

        </doctrine-mapping>

    .. code-block:: yaml

        Application\Model\ArticleAttribute:
          type: entity
          id:
            article:
              associationKey: true
            attribute:
              type: string
          fields:
            value:
              type: string
          manyToOne:
            article:
              targetEntity: Article
              inversedBy: attributes


用例2：简单派生标识
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

有时你需要两个对象通过一对一关联相关，并且依赖类应该重用它所依赖的类的主键。
一个很好的例子是用户-地址关系：

.. configuration-block::

    .. code-block:: php

        <?php
        /**
         * @Entity
         */
        class User
        {
            /** @Id @Column(type="integer") @GeneratedValue */
            private $id;
        }

        /**
         * @Entity
         */
        class Address
        {
            /** @Id @OneToOne(targetEntity="User") */
            private $user;
        }

    .. code-block:: yaml

        User:
          type: entity
          id:
            id:
              type: integer
              generator:
                strategy: AUTO

        Address:
          type: entity
          id:
            user:
              associationKey: true
          oneToOne:
            user:
              targetEntity: User


用例3：使用元数据连接表
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

在经典订单产品商店示例中，订单商品的概念包含对订单和产品的引用以及其他数据，例如购买的产品数量，甚至可能是当前价格。

.. code-block:: php

    use Doctrine\Common\Collections\ArrayCollection;

    /** @Entity */
    class Order
    {
        /** @Id @Column(type="integer") @GeneratedValue */
        private $id;

        /** @ManyToOne(targetEntity="Customer") */
        private $customer;
        /** @OneToMany(targetEntity="OrderItem", mappedBy="order") */
        private $items;

        /** @Column(type="boolean") */
        private $payed = false;
        /** @Column(type="boolean") */
        private $shipped = false;
        /** @Column(type="datetime") */
        private $created;

        public function __construct(Customer $customer)
        {
            $this->customer = $customer;
            $this->items = new ArrayCollection();
            $this->created = new \DateTime("now");
        }
    }

    /** @Entity */
    class Product
    {
        /** @Id @Column(type="integer") @GeneratedValue */
        private $id;

        /** @Column(type="string") */
        private $name;

        /** @Column(type="decimal") */
        private $currentPrice;

        public function getCurrentPrice()
        {
            return $this->currentPrice;
        }
    }

    /** @Entity */
    class OrderItem
    {
        /** @Id @ManyToOne(targetEntity="Order") */
        private $order;

        /** @Id @ManyToOne(targetEntity="Product") */
        private $product;

        /** @Column(type="integer") */
        private $amount = 1;

        /** @Column(type="decimal") */
        private $offeredPrice;

        public function __construct(Order $order, Product $product, $amount = 1)
        {
            $this->order = $order;
            $this->product = $product;
            $this->offeredPrice = $product->getCurrentPrice();
        }
    }

性能事项
~~~~~~~~~~~~~~~~~~~~~~~~~~

与使用具有简单代理(surrogate)键的实体相比，使用复合键总是会带来性能损失。
这种性能影响主要是由于处理这种键所需的额外PHP代码，最明显的是在使用派生标识符时。

在SQL端，没有太多开销，因为不必执行额外或意外的查询来管理具有派生外键的实体。
