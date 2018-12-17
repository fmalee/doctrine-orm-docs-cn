使用可嵌入类来分离关注点
-------------------------------------

可嵌入(Embeddable)类本身不是实体，但可以嵌入到实体中，也可以在DQL中进行查询。
你通常希望使用它们来减少重复或分离关注点。日期范围或地址等值对象是此功能的主要用例。

.. note::

    可嵌入类只能包含具有基本 ``@Column`` 映射的属性。

出于本教程的目的，我们假设你的应用中有一个 ``User`` 类，并且你希望在类中存储一个地址。
我们将 ``Address`` 类建模为一个可嵌入类，而不是简单地将相应的列添加到 ``User`` 类中。

.. configuration-block::

    .. code-block:: php

        <?php

        /** @Entity */
        class User
        {
            /** @Embedded(class = "Address") */
            private $address;
        }

        /** @Embeddable */
        class Address
        {
            /** @Column(type = "string") */
            private $street;

            /** @Column(type = "string") */
            private $postalCode;

            /** @Column(type = "string") */
            private $city;

            /** @Column(type = "string") */
            private $country;
        }

    .. code-block:: xml

        <doctrine-mapping>
            <entity name="User">
                <embedded name="address" class="Address" />
            </entity>

            <embeddable name="Address">
                <field name="street" type="string" />
                <field name="postalCode" type="string" />
                <field name="city" type="string" />
                <field name="country" type="string" />
            </embeddable>
        </doctrine-mapping>

    .. code-block:: yaml

        User:
          type: entity
          embedded:
            address:
              class: Address

        Address:
          type: embeddable
          fields:
            street: { type: string }
            postalCode: { type: string }
            city: { type: string }
            country: { type: string }

就数据库模式而言，Doctrine会自动将 ``Address`` 类中的所有列内联到 ``User`` 类的表中，就像你直接在那里声明它们一样。

初始化可嵌入类
------------------------

如果可嵌入类中的所有字段都是 ``nullable``，则可能需要初始化该可嵌入类，以避免获取一个 ``null`` 值而不是嵌入对象。

.. code-block:: php

    public function __construct()
    {
        $this->address = new Address();
    }

列前缀
----------------

默认情况下，Doctrine使用值对象的名称为列添加前缀来命名该列。

按照上面的示例，你的列将命名为 ``address_street``、``address_postalCode`` ...

你可以通过更改 ``@Embedded`` 注释中的 ``columnPrefix`` 属性来更改此行为以满足你的需要。

以下示例展示了如何将前缀设置为 ``myPrefix_``：

.. configuration-block::

    .. code-block:: php

        <?php

        /** @Entity */
        class User
        {
            /** @Embedded(class = "Address", columnPrefix = "myPrefix_") */
            private $address;
        }

    .. code-block:: xml

        <entity name="User">
            <embedded name="address" class="Address" column-prefix="myPrefix_" />
        </entity>

    .. code-block:: yaml

        User:
          type: entity
          embedded:
            address:
              class: Address
              columnPrefix: myPrefix_

要让Doctrine删除前缀并直接使用值对象的属性名，请设置为 ``columnPrefix=false`` （对于XML：``use-column-prefix="false"``）：

.. configuration-block::

    .. code-block:: php

        <?php

        /** @Entity */
        class User
        {
            /** @Embedded(class = "Address", columnPrefix = false) */
            private $address;
        }

    .. code-block:: yaml

        User:
          type: entity
          embedded:
            address:
              class: Address
              columnPrefix: false

    .. code-block:: xml

        <entity name="User">
            <embedded name="address" class="Address" use-column-prefix="false" />
        </entity>


DQL
---

你还可以在DQL查询中使用嵌入类的映射字段，就像已经在 ``User`` 类中声明了它们一样：

.. code-block:: sql

    SELECT u FROM User u WHERE u.address.city = :myCity
