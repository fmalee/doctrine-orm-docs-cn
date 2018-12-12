关联映射
===================

本章介绍了对象之间的映射关联。

你将始终使用对象的引用而不是在代码中使用外键，而Doctrine将在内部将这些引用转换为外键。

- 一个由外键代表的对单个对象的引用。
- 一个由指向持有集合的对象的许多外键代表的对象集合

本章分为三个不同的部分。

- 给出了所有可能的关联映射的用例的列表。
- 解释了 :ref:`association_mapping_defaults`，简化了示例。
- 引入了包含关联实体的 :ref:`collections`。

处理关联关系的一个技巧是从 **左** 到 **右** 读取关系，其中 *左* 这个词指的是当前实体。例如：

- ``OneToMany`` - 当前实体的 *一个* 实例具有 *许多* 指向已引用(refered)实体的实例（引用）。
- ``ManyToOne`` - 当前实体的 *许多* 实例引用已引用(refered)实体的 *一个* 实例。
- ``OneToOne`` - 当前实体的 *一个* 实例引用已引用(refered)实体的 *一个* 实例。

请参阅下面的所有可能的关联关系。

如果 *仅* 一个关联的一方具有指向另一方的属性，则认为此关联是 **单向** 的。

要充分了解关联，你还应该阅读 :doc:`关联的拥有方和从属方 <unitofwork-associations>`

单向多对一
---------------------------

**多对一** 关联是对象之间最常见的关联。*许多* ``User`` 拥有 *一个* ``Address`` 的示例：

.. configuration-block::

    .. code-block:: php

        <?php
        /** @Entity */
        class User
        {
            // ...

            /**
             * @ManyToOne(targetEntity="Address")
             * @JoinColumn(name="address_id", referencedColumnName="id")
             */
            private $address;
        }

        /** @Entity */
        class Address
        {
            // ...
        }

    .. code-block:: xml

        <doctrine-mapping>
            <entity name="User">
                <many-to-one field="address" target-entity="Address">
                    <join-column name="address_id" referenced-column-name="id" />
                </many-to-one>
            </entity>
        </doctrine-mapping>

    .. code-block:: yaml

        User:
          type: entity
          manyToOne:
            address:
              targetEntity: Address
              joinColumn:
                name: address_id
                referencedColumnName: id


.. note::

    上面的 ``@JoinColumn`` 是可选的，因为它始终默认为
    ``address_id`` 和 ``id``。所以你可以省略它以使用默认值。

生成的MySQL模式：

.. code-block:: sql

    CREATE TABLE User (
        id INT AUTO_INCREMENT NOT NULL,
        address_id INT DEFAULT NULL,
        PRIMARY KEY(id)
    ) ENGINE = InnoDB;

    CREATE TABLE Address (
        id INT AUTO_INCREMENT NOT NULL,
        PRIMARY KEY(id)
    ) ENGINE = InnoDB;

    ALTER TABLE User ADD FOREIGN KEY (address_id) REFERENCES Address(id);

单向一对一
--------------------------

以下是一个 ``Product`` 实体引用一个 ``Shipment`` 实体的 **一对一** 关联的示例。

.. configuration-block::

    .. code-block:: php

        <?php
        /** @Entity */
        class Product
        {
            // ...

            /**
             * One Product has One Shipment.
             * @OneToOne(targetEntity="Shipment")
             * @JoinColumn(name="shipment_id", referencedColumnName="id")
             */
            private $shipment;

            // ...
        }

        /** @Entity */
        class Shipment
        {
            // ...
        }

    .. code-block:: xml

        <doctrine-mapping>
            <entity class="Product">
                <one-to-one field="shipment" target-entity="Shipment">
                    <join-column name="shipment_id" referenced-column-name="id" />
                </one-to-one>
            </entity>
        </doctrine-mapping>

    .. code-block:: yaml

        Product:
          type: entity
          oneToOne:
            shipment:
              targetEntity: Shipment
              joinColumn:
                name: shipment_id
                referencedColumnName: id

请注意，在此示例中并不真正需要 ``@JoinColumn``，因为该注释与默认值是相同的。

生成的MySQL模式：

.. code-block:: sql

    CREATE TABLE Product (
        id INT AUTO_INCREMENT NOT NULL,
        shipment_id INT DEFAULT NULL,
        UNIQUE INDEX UNIQ_6FBC94267FE4B2B (shipment_id),
        PRIMARY KEY(id)
    ) ENGINE = InnoDB;
    CREATE TABLE Shipment (
        id INT AUTO_INCREMENT NOT NULL,
        PRIMARY KEY(id)
    ) ENGINE = InnoDB;
    ALTER TABLE Product ADD FOREIGN KEY (shipment_id) REFERENCES Shipment(id);

双向一对一
-------------------------

这是一个 ``Customer`` 实体和一个 ``Cart`` 实体之间的 **一对一** 关系。
``Cart`` 同时具有一个 ``Customer`` 的引用，所以该关系是双向的。

在这里，我们第一次看到 ``mappedBy`` 和 ``inversedBy`` 注释。
它们用于告诉Doctrine另一方的哪个属性引用该对象。

.. configuration-block::

    .. code-block:: php

        <?php
        /** @Entity */
        class Customer
        {
            // ...

            /**
             * One Customer has One Cart.
             * @OneToOne(targetEntity="Cart", mappedBy="customer")
             */
            private $cart;

            // ...
        }

        /** @Entity */
        class Cart
        {
            // ...

            /**
             * One Cart has One Customer.
             * @OneToOne(targetEntity="Customer", inversedBy="cart")
             * @JoinColumn(name="customer_id", referencedColumnName="id")
             */
            private $customer;

            // ...
        }

    .. code-block:: xml

        <doctrine-mapping>
            <entity name="Customer">
                <one-to-one field="cart" target-entity="Cart" mapped-by="customer" />
            </entity>
            <entity name="Cart">
                <one-to-one field="customer" target-entity="Customer" inversed-by="cart">
                    <join-column name="customer_id" referenced-column-name="id" />
                </one-to-one>
            </entity>
        </doctrine-mapping>

    .. code-block:: yaml

        Customer:
          oneToOne:
            cart:
              targetEntity: Cart
              mappedBy: customer
        Cart:
          oneToOne:
            customer:
              targetEntity: Customer
              inversedBy: cart
              joinColumn:
                name: customer_id
                referencedColumnName: id

请注意，在此示例中并不真正需要 ``@JoinColumn``，因为该注释与默认值是相同的。

生成的MySQL模式：

.. code-block:: sql

    CREATE TABLE Cart (
        id INT AUTO_INCREMENT NOT NULL,
        customer_id INT DEFAULT NULL,
        PRIMARY KEY(id)
    ) ENGINE = InnoDB;
    CREATE TABLE Customer (
        id INT AUTO_INCREMENT NOT NULL,
        PRIMARY KEY(id)
    ) ENGINE = InnoDB;
    ALTER TABLE Cart ADD FOREIGN KEY (customer_id) REFERENCES Customer(id);

我们可以选择将 ``inversedBy`` 属性放置在哪一方。
因为它放置在 ``Cart`` 类上，则意味着它是关系的拥有方，因此持有外键。

自引用的一对一
----------------------------

你可以定义一个 *自引用* 的 **一对一** 关系，如下所示：

.. code-block:: php

    /** @Entity */
    class Student
    {
        // ...

        /**
         * One Student has One Student.
         * @OneToOne(targetEntity="Student")
         * @JoinColumn(name="mentor_id", referencedColumnName="id")
         */
        private $mentor;

        // ...
    }

请注意，在此示例中并不真正需要 ``@JoinColumn``，因为该注释与默认值是相同的。

生成的MySQL模式：

.. code-block:: sql

    CREATE TABLE Student (
        id INT AUTO_INCREMENT NOT NULL,
        mentor_id INT DEFAULT NULL,
        PRIMARY KEY(id)
    ) ENGINE = InnoDB;
    ALTER TABLE Student ADD FOREIGN KEY (mentor_id) REFERENCES Student(id);

双向一对多
--------------------------

除非你使用了一个连接表，否则 **一对多** 关联 *必须* 是 *双向* 的。
这是因为一对多关联中的 *多* 方持有外键，从而使其成为拥有方。Doctrine需要定义 *多* 方以理解此关联。

这种 **双向** 映射在 *一* 方需要 ``mappedBy`` 属性，在 *多* 方需要 ``inversedBy`` 属性。

这意味着 **双向一对多** 和 **双向多对一** 之间 *没有* 区别的。

.. configuration-block::

    .. code-block:: php

        <?php
        use Doctrine\Common\Collections\ArrayCollection;

        /** @Entity */
        class Product
        {
            // ...
            /**
             * One Product has Many Features.
             * @OneToMany(targetEntity="Feature", mappedBy="product")
             */
            private $features;
            // ...

            public function __construct() {
                $this->features = new ArrayCollection();
            }
        }

        /** @Entity */
        class Feature
        {
            // ...
            /**
             * Many Features have One Product.
             * @ManyToOne(targetEntity="Product", inversedBy="features")
             * @JoinColumn(name="product_id", referencedColumnName="id")
             */
            private $product;
            // ...
        }

    .. code-block:: xml

        <doctrine-mapping>
            <entity name="Product">
                <one-to-many field="features" target-entity="Feature" mapped-by="product" />
            </entity>
            <entity name="Feature">
                <many-to-one field="product" target-entity="Product" inversed-by="features">
                    <join-column name="product_id" referenced-column-name="id" />
                </many-to-one>
            </entity>
        </doctrine-mapping>

    .. code-block:: yaml

        Product:
          type: entity
          oneToMany:
            features:
              targetEntity: Feature
              mappedBy: product
        Feature:
          type: entity
          manyToOne:
            product:
              targetEntity: Product
              inversedBy: features
              joinColumn:
                name: product_id
                referencedColumnName: id

请注意，在此示例中并不真正需要 ``@JoinColumn``，因为该注释与默认值是相同的。

生成的MySQL模式：

.. code-block:: sql

    CREATE TABLE Product (
        id INT AUTO_INCREMENT NOT NULL,
        PRIMARY KEY(id)
    ) ENGINE = InnoDB;
    CREATE TABLE Feature (
        id INT AUTO_INCREMENT NOT NULL,
        product_id INT DEFAULT NULL,
        PRIMARY KEY(id)
    ) ENGINE = InnoDB;
    ALTER TABLE Feature ADD FOREIGN KEY (product_id) REFERENCES Product(id);

使用连接表的单向一对多
-------------------------------------------

可以通过一个 *连接表* 来映射 **单向一对多** 关联。
从Doctrine的角度来看，它被简单地映射为 **单向多对多**，其中一个连接列上的唯一约束强制执行一对多基数(cardinality)。

以下示例设置了这种单向一对多关联：

.. configuration-block::

    .. code-block:: php

        <?php
        /** @Entity */
        class User
        {
            // ...

            /**
             * Many User have Many Phonenumbers.
             * @ManyToMany(targetEntity="Phonenumber")
             * @JoinTable(name="users_phonenumbers",
             *      joinColumns={@JoinColumn(name="user_id", referencedColumnName="id")},
             *      inverseJoinColumns={@JoinColumn(name="phonenumber_id", referencedColumnName="id", unique=true)}
             *      )
             */
            private $phonenumbers;

            public function __construct()
            {
                $this->phonenumbers = new \Doctrine\Common\Collections\ArrayCollection();
            }

            // ...
        }

        /** @Entity */
        class Phonenumber
        {
            // ...
        }

    .. code-block:: xml

        <doctrine-mapping>
            <entity name="User">
                <many-to-many field="phonenumbers" target-entity="Phonenumber">
                    <join-table name="users_phonenumbers">
                        <join-columns>
                            <join-column name="user_id" referenced-column-name="id" />
                        </join-columns>
                        <inverse-join-columns>
                            <join-column name="phonenumber_id" referenced-column-name="id" unique="true" />
                        </inverse-join-columns>
                    </join-table>
                </many-to-many>
            </entity>
        </doctrine-mapping>

    .. code-block:: yaml

        User:
          type: entity
          manyToMany:
            phonenumbers:
              targetEntity: Phonenumber
              joinTable:
                name: users_phonenumbers
                joinColumns:
                  user_id:
                    referencedColumnName: id
                inverseJoinColumns:
                  phonenumber_id:
                    referencedColumnName: id
                    unique: true

生成的MySQL模式：

.. code-block:: sql

    CREATE TABLE User (
        id INT AUTO_INCREMENT NOT NULL,
        PRIMARY KEY(id)
    ) ENGINE = InnoDB;

    CREATE TABLE users_phonenumbers (
        user_id INT NOT NULL,
        phonenumber_id INT NOT NULL,
        UNIQUE INDEX users_phonenumbers_phonenumber_id_uniq (phonenumber_id),
        PRIMARY KEY(user_id, phonenumber_id)
    ) ENGINE = InnoDB;

    CREATE TABLE Phonenumber (
        id INT AUTO_INCREMENT NOT NULL,
        PRIMARY KEY(id)
    ) ENGINE = InnoDB;

    ALTER TABLE users_phonenumbers ADD FOREIGN KEY (user_id) REFERENCES User(id);
    ALTER TABLE users_phonenumbers ADD FOREIGN KEY (phonenumber_id) REFERENCES Phonenumber(id);

自引用的一对多
-----------------------------

你还可以设置 *自引用* 的 **一对多** 关联。
在此示例中，我们通过创建一个自引用关系来设置 ``Category`` 对象的层级。
这有效地模拟了类别的层级，而从数据库的角度来看，这被称为一个 **邻接表方法**。

.. configuration-block::

    .. code-block:: php

        <?php
        /** @Entity */
        class Category
        {
            // ...
            /**
             * One Category has Many Categories.
             * @OneToMany(targetEntity="Category", mappedBy="parent")
             */
            private $children;

            /**
             * Many Categories have One Category.
             * @ManyToOne(targetEntity="Category", inversedBy="children")
             * @JoinColumn(name="parent_id", referencedColumnName="id")
             */
            private $parent;
            // ...

            public function __construct() {
                $this->children = new \Doctrine\Common\Collections\ArrayCollection();
            }
        }

    .. code-block:: xml

        <doctrine-mapping>
            <entity name="Category">
                <one-to-many field="children" target-entity="Category" mapped-by="parent" />
                <many-to-one field="parent" target-entity="Category" inversed-by="children" />
            </entity>
        </doctrine-mapping>

    .. code-block:: yaml

        Category:
          type: entity
          oneToMany:
            children:
              targetEntity: Category
              mappedBy: parent
          manyToOne:
            parent:
              targetEntity: Category
              inversedBy: children

请注意，在此示例中并不真正需要 ``@JoinColumn``，因为该注释与默认值是相同的。

生成的MySQL模式：

.. code-block:: sql

    CREATE TABLE Category (
        id INT AUTO_INCREMENT NOT NULL,
        parent_id INT DEFAULT NULL,
        PRIMARY KEY(id)
    ) ENGINE = InnoDB;
    ALTER TABLE Category ADD FOREIGN KEY (parent_id) REFERENCES Category(id);

单向多对多
----------------------------

真正的多对多关联不太常见。以下示例展示了 ``User`` 和 ``Group`` 实体之间的 **单向** 关联：

.. configuration-block::

    .. code-block:: php

        <?php
        /** @Entity */
        class User
        {
            // ...

            /**
             * Many Users have Many Groups.
             * @ManyToMany(targetEntity="Group")
             * @JoinTable(name="users_groups",
             *      joinColumns={@JoinColumn(name="user_id", referencedColumnName="id")},
             *      inverseJoinColumns={@JoinColumn(name="group_id", referencedColumnName="id")}
             *      )
             */
            private $groups;

            // ...

            public function __construct() {
                $this->groups = new \Doctrine\Common\Collections\ArrayCollection();
            }
        }

        /** @Entity */
        class Group
        {
            // ...
        }

    .. code-block:: xml

        <doctrine-mapping>
            <entity name="User">
                <many-to-many field="groups" target-entity="Group">
                    <join-table name="users_groups">
                        <join-columns>
                            <join-column name="user_id" referenced-column-name="id" />
                        </join-columns>
                        <inverse-join-columns>
                            <join-column name="group_id" referenced-column-name="id" />
                        </inverse-join-columns>
                    </join-table>
                </many-to-many>
            </entity>
        </doctrine-mapping>

    .. code-block:: yaml

        User:
          type: entity
          manyToMany:
            groups:
              targetEntity: Group
              joinTable:
                name: users_groups
                joinColumns:
                  user_id:
                    referencedColumnName: id
                inverseJoinColumns:
                  group_id:
                    referencedColumnName: id

生成的MySQL模式：

.. code-block:: sql

    CREATE TABLE User (
        id INT AUTO_INCREMENT NOT NULL,
        PRIMARY KEY(id)
    ) ENGINE = InnoDB;
    CREATE TABLE users_groups (
        user_id INT NOT NULL,
        group_id INT NOT NULL,
        PRIMARY KEY(user_id, group_id)
    ) ENGINE = InnoDB;
    CREATE TABLE Group (
        id INT AUTO_INCREMENT NOT NULL,
        PRIMARY KEY(id)
    ) ENGINE = InnoDB;
    ALTER TABLE users_groups ADD FOREIGN KEY (user_id) REFERENCES User(id);
    ALTER TABLE users_groups ADD FOREIGN KEY (group_id) REFERENCES Group(id);

.. note::

    为什么多对多关联不太常见？因为你经常要关联其他属性到一个关联，所以在这种情况下，你会引入一个关联类。
    因此，*直接* 的多对多关联消失，并被 *3* 个参与类之间的一对多/多对一关联所取代。

双向多对多
---------------------------

这是一个类似多对多的关系，除了它是双向的。

.. configuration-block::

    .. code-block:: php

        <?php
        /** @Entity */
        class User
        {
            // ...

            /**
             * Many Users have Many Groups.
             * @ManyToMany(targetEntity="Group", inversedBy="users")
             * @JoinTable(name="users_groups")
             */
            private $groups;

            public function __construct() {
                $this->groups = new \Doctrine\Common\Collections\ArrayCollection();
            }

            // ...
        }

        /** @Entity */
        class Group
        {
            // ...
            /**
             * Many Groups have Many Users.
             * @ManyToMany(targetEntity="User", mappedBy="groups")
             */
            private $users;

            public function __construct() {
                $this->users = new \Doctrine\Common\Collections\ArrayCollection();
            }

            // ...
        }

    .. code-block:: xml

        <doctrine-mapping>
            <entity name="User">
                <many-to-many field="groups" inversed-by="users" target-entity="Group">
                    <join-table name="users_groups">
                        <join-columns>
                            <join-column name="user_id" referenced-column-name="id" />
                        </join-columns>
                        <inverse-join-columns>
                            <join-column name="group_id" referenced-column-name="id" />
                        </inverse-join-columns>
                    </join-table>
                </many-to-many>
            </entity>

            <entity name="Group">
                <many-to-many field="users" mapped-by="groups" target-entity="User"/>
            </entity>
        </doctrine-mapping>

    .. code-block:: yaml

        User:
          type: entity
          manyToMany:
            groups:
              targetEntity: Group
              inversedBy: users
              joinTable:
                name: users_groups
                joinColumns:
                  user_id:
                    referencedColumnName: id
                inverseJoinColumns:
                  group_id:
                    referencedColumnName: id

        Group:
          type: entity
          manyToMany:
            users:
              targetEntity: User
              mappedBy: groups

MySQL的模式与上面的 *单向多对多* 示例完全相同。

多对多关联的拥有方和从属方
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

对于 **多对多** 关联时，你可以选择哪个实体是 *拥有* 方，哪个实体是 *从属* 方。
从开发人员的角度来看，有一个非常简单的语义规则来决定哪一方更适合作为 *拥有* 方。
你只需要问自己负责连接管理的是哪个实体，然后选择它作为 *拥有* 方。

就拿 ``Article`` 和 ``Tag`` 这两个实体作为示例。
无论何时你都是想要将一个 ``Article`` 连接到一个 ``Tag``，反之亦然，主要是 ``Article`` 负责这种关系。
每当你添加新文章时，你都希望将其与现有标签或新标签相关联。
你的“创建文章”表单可能会支持此概念并允许直接指定标签。
这就是为什么你应该选择 ``Article`` 作为 *拥有* 方，因为它使代码更容易理解：

.. code-block:: php

    class Article
    {
        private $tags;

        public function addTag(Tag $tag)
        {
            $tag->addArticle($this); // 同步更新从属方
            $this->tags[] = $tag;
        }
    }

    class Tag
    {
        private $articles;

        public function addArticle(Article $article)
        {
            $this->articles[] = $article;
        }
    }

这将允许在关联的 ``Article`` 方对标签添加进行分组：

.. code-block:: php

    $article = new Article();
    $article->addTag($tagA);
    $article->addTag($tagB);

自引用的多对多
------------------------------

你甚至可以拥有一个 *自引用* 的 **多对多** 关联。
一个常见的场景是，一个 ``User`` 有好友关系，而该关系的目标实体是一个 ``User``，因此它是自引用。

在这个例子中，该关联是双向的，因此一个 ``User`` 有一个名为 ``$friendsWithMe`` 的字段和 ``$myFriends`` 关系。

.. code-block:: php

    /** @Entity */
    class User
    {
        // ...

        /**
         * Many Users have Many Users.
         * @ManyToMany(targetEntity="User", mappedBy="myFriends")
         */
        private $friendsWithMe;

        /**
         * Many Users have many Users.
         * @ManyToMany(targetEntity="User", inversedBy="friendsWithMe")
         * @JoinTable(name="friends",
         *      joinColumns={@JoinColumn(name="user_id", referencedColumnName="id")},
         *      inverseJoinColumns={@JoinColumn(name="friend_user_id", referencedColumnName="id")}
         *      )
         */
        private $myFriends;

        public function __construct() {
            $this->friendsWithMe = new \Doctrine\Common\Collections\ArrayCollection();
            $this->myFriends = new \Doctrine\Common\Collections\ArrayCollection();
        }

        // ...
    }

生成的MySQL模式：

.. code-block:: sql

    CREATE TABLE User (
        id INT AUTO_INCREMENT NOT NULL,
        PRIMARY KEY(id)
    ) ENGINE = InnoDB;
    CREATE TABLE friends (
        user_id INT NOT NULL,
        friend_user_id INT NOT NULL,
        PRIMARY KEY(user_id, friend_user_id)
    ) ENGINE = InnoDB;
    ALTER TABLE friends ADD FOREIGN KEY (user_id) REFERENCES User(id);
    ALTER TABLE friends ADD FOREIGN KEY (friend_user_id) REFERENCES User(id);

.. _association_mapping_defaults:

映射的默认值
----------------

``@JoinColumn`` 和 ``@JoinTable`` 定义通常是可选的，并且具有合理的默认值。
一对一/多对一关联中的连接列的默认值如下：

::

    name: "<fieldname>_id"
    referencedColumnName: "id"

作为示例，请思考以下映射：

.. configuration-block::

    .. code-block:: php

        <?php
        /** @OneToOne(targetEntity="Shipment") */
        private $shipment;

    .. code-block:: xml

        <doctrine-mapping>
            <entity class="Product">
                <one-to-one field="shipment" target-entity="Shipment" />
            </entity>
        </doctrine-mapping>

    .. code-block:: yaml

        Product:
          type: entity
          oneToOne:
            shipment:
              targetEntity: Shipment

这与以下更详细的映射基本相同：

.. configuration-block::

    .. code-block:: php

        <?php
        /**
         * One Product has One Shipment.
         * @OneToOne(targetEntity="Shipment")
         * @JoinColumn(name="shipment_id", referencedColumnName="id")
         */
        private $shipment;

    .. code-block:: xml

        <doctrine-mapping>
            <entity class="Product">
                <one-to-one field="shipment" target-entity="Shipment">
                    <join-column name="shipment_id" referenced-column-name="id" />
                </one-to-one>
            </entity>
        </doctrine-mapping>

    .. code-block:: yaml

        Product:
          type: entity
          oneToOne:
            shipment:
              targetEntity: Shipment
              joinColumn:
                name: shipment_id
                referencedColumnName: id

用于 **多对多** 映射的 ``@JoinTable`` 定义具有类似的默认值。作为示例，请思考以下映射：

.. configuration-block::

    .. code-block:: php

        <?php
        class User
        {
            //...
            /** @ManyToMany(targetEntity="Group") */
            private $groups;
            //...
        }

    .. code-block:: xml

        <doctrine-mapping>
            <entity class="User">
                <many-to-many field="groups" target-entity="Group" />
            </entity>
        </doctrine-mapping>

    .. code-block:: yaml

        User:
          type: entity
          manyToMany:
            groups:
              targetEntity: Group

这与以下更详细的映射基本相同：

.. configuration-block::

    .. code-block:: php

        <?php
        class User
        {
            //...
            /**
             * Many Users have Many Groups.
             * @ManyToMany(targetEntity="Group")
             * @JoinTable(name="User_Group",
             *      joinColumns={@JoinColumn(name="User_id", referencedColumnName="id")},
             *      inverseJoinColumns={@JoinColumn(name="Group_id", referencedColumnName="id")}
             *      )
             */
            private $groups;
            //...
        }

    .. code-block:: xml

        <doctrine-mapping>
            <entity class="User">
                <many-to-many field="groups" target-entity="Group">
                    <join-table name="User_Group">
                        <join-columns>
                            <join-column id="User_id" referenced-column-name="id" />
                        </join-columns>
                        <inverse-join-columns>
                            <join-column id="Group_id" referenced-column-name="id" />
                        </inverse-join-columns>
                    </join-table>
                </many-to-many>
            </entity>
        </doctrine-mapping>

    .. code-block:: yaml

        User:
          type: entity
          manyToMany:
            groups:
              targetEntity: Group
              joinTable:
                name: User_Group
                joinColumns:
                  User_id:
                    referencedColumnName: id
                inverseJoinColumns:
                  Group_id:
                    referencedColumnName: id

在这个例子中，连接表的名称默认为参与类的简单、非限定类名称的组合，并用下划线（``_``）字符分隔。
连接列的名称默认为目标类的简单、非限定类名，在附加 ``_id``。
``referencedColumnName`` 始终默认为 ``id``，就像在一对一或多对一映射中一样。

如果你接受了这些默认值，则可以将映射代码减少到最小。

.. _collections:

集合
-----------

PHP数组大多数情况下都很好用，但不幸的是，缺少使它们适合在ORM上下文中进行延迟加载的功能。
这就是为什么在本手册中的多值关联的所有示例中，我们使用了一个 ``ArrayCollection``。
该默认实现了 ``Collection`` 接口，并且这两个类都定义在 ``Doctrine\Common\Collections`` 命名空间下。
一个集合实现了PHP的 ``ArrayAccess``、``Traversable`` 以及 ``Countable`` 接口.

.. note::

    ``Collection`` 接口和 ``ArrayCollection``
    类与Doctrine命名空间中的其他所有类一样，既不是 ``ORM`` 的一部分，也不是 ``DBAL``
    的一部分，它们是一个普通的PHP类，除了依赖PHP本身（和 ``SPL``）之外没有外部依赖。
    因此，在你的模型和其他地方使用此类时不会产生与 ``ORM`` 的耦合。

初始化集合
------------------------

你应该始终在实体的构造函数中初始化你的 ``@OneToMany`` 和 ``@ManyToMany`` 关联的集合：

.. code-block:: php

    use Doctrine\Common\Collections\Collection;
    use Doctrine\Common\Collections\ArrayCollection;

    /** @Entity */
    class User
    {
        /**
         * Many Users have Many Groups.
         * @var Collection
         * @ManyToMany(targetEntity="Group")
         */
        private $groups;

        public function __construct()
        {
            $this->groups = new ArrayCollection();
        }

        public function getGroups()
        {
            return $this->groups;
        }
    }

即使该实体尚未与一个 ``EntityManager`` 关联，以下代码也将起作用：

.. code-block:: php

    <?php
    $group = new Group();
    $user = new User();
    $user->getGroups()->add($group);
