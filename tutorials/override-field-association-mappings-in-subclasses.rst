重写子类中的字段关联映射
-------------------------------------------------

有时要持久化实体但需要重写全部或部分映射元数据。
有时候，重写的映射来自使用 **trait** 的实体，而 *trait* 本身具有映射元数据。
本教程将介绍如何重写映射元数据，特别是属性和关联元数据。
此处的示例显示了使用一个trait的类的重写，但如本教程末尾所示，在继承基类时也适用此重写示例。

假设我们有一个 ``ExampleEntityWithOverride`` 类。这个类使用 ``ExampleTrait`` trait：

.. code-block:: php

    <?php
    /**
     * @Entity
     *
     * @AttributeOverrides({
     *      @AttributeOverride(name="foo",
     *          column=@Column(
     *              name     = "foo_overridden",
     *              type     = "integer",
     *              length   = 140,
     *              nullable = false,
     *              unique   = false
     *          )
     *      )
     * })
     *
     * @AssociationOverrides({
     *      @AssociationOverride(name="bar",
     *          joinColumns=@JoinColumn(
     *              name="example_entity_overridden_bar_id", referencedColumnName="id"
     *          )
     *      )
     * })
     */
    class ExampleEntityWithOverride
    {
        use ExampleTrait;
    }

    /**
     * @Entity
     */
    class Bar
    {
        /** @Id @Column(type="string") */
        private $id;
    }

该文档区块展示了属性和关联类型的元数据重写。
它基本上更改了映射到 ``foo`` 属性的列的名称以及与上面显示的 ``Bar`` 类相关的 ``bar`` 关联。
这是具有映射元数据的 ``trait``，该元数据被上面的注释重写：

.. code-block:: php

    <?php
    /**
     * Trait class
     */
    trait ExampleTrait
    {
        /** @Id @Column(type="string") */
        private $id;

        /**
         * @Column(name="trait_foo", type="integer", length=100, nullable=true, unique=true)
         */
        protected $foo;

        /**
         * @OneToOne(targetEntity="Bar", cascade={"persist", "merge"})
         * @JoinColumn(name="example_trait_bar_id", referencedColumnName="id")
         */
        protected $bar;
    }

只是继承一个类的情况也是一样的：

.. code-block:: php

    <?php
    class ExampleEntityWithOverride extends BaseEntityWithSomeMapping
    {
        // ...
    }

XML和YAML（:ref:`示例 <inheritence_mapping_overrides>`）也支持重写。
