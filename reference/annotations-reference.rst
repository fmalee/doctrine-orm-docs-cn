注释参考
=====================

你可能已经以某种形式使用了文档块注释，最有可能为
``PHPDocumentor`` （``@author``、``@link``...）等工具提供文档元数据。
文档区块注释是一种在文档部分中嵌入元数据的工具，然后可以通过某些工具进行处理。
Doctrine2归纳了文档区块注释的概念，以便它们可以用于任何类型的元数据，因此很容易定义新的文档区块注释。
为了允许涉及更多的注释值并减少与其他文档区块注释发生冲突的可能性，Doctrine2文档区块注释使用另一种语法，该语法受Java5中引入的注释语法的启发。

这些增强的文档区块注释的实现位于 ``Doctrine\Common\Annotations`` 命名空间中，因此是 ``Common`` 软件包的一部分。
Doctrine2文档区块注释支持命名空间和嵌套注释等功能。
Doctrine2 ORM定义了自己的一组文档区块注释，用于提供对象-关系映射的元数据。

.. note::

    如果你对文档区块注释的概念不满意，请不要担心。
    如前所述，Doctrine2提供了XML和YAML替代方案，你可以轻松实现自己喜欢的机制来定义ORM元数据。

在本章中，每个Doctrine2注释的参考文献都给出了对其上下文和用法的简短解释。

索引
-----

-  :ref:`@Column <annref_column>`
-  :ref:`@ColumnResult <annref_column_result>`
-  :ref:`@Cache <annref_cache>`
-  :ref:`@ChangeTrackingPolicy <annref_changetrackingpolicy>`
-  :ref:`@CustomIdGenerator <annref_customidgenerator>`
-  :ref:`@DiscriminatorColumn <annref_discriminatorcolumn>`
-  :ref:`@DiscriminatorMap <annref_discriminatormap>`
-  :ref:`@Embeddable <annref_embeddable>`
-  :ref:`@Embedded <annref_embedded>`
-  :ref:`@Entity <annref_entity>`
-  :ref:`@EntityResult <annref_entity_result>`
-  :ref:`@FieldResult <annref_field_result>`
-  :ref:`@GeneratedValue <annref_generatedvalue>`
-  :ref:`@HasLifecycleCallbacks <annref_haslifecyclecallbacks>`
-  :ref:`@Index <annref_index>`
-  :ref:`@Id <annref_id>`
-  :ref:`@InheritanceType <annref_inheritancetype>`
-  :ref:`@JoinColumn <annref_joincolumn>`
-  :ref:`@JoinColumns <annref_joincolumns>`
-  :ref:`@JoinTable <annref_jointable>`
-  :ref:`@ManyToOne <annref_manytoone>`
-  :ref:`@ManyToMany <annref_manytomany>`
-  :ref:`@MappedSuperclass <annref_mappedsuperclass>`
-  :ref:`@NamedNativeQuery <annref_named_native_query>`
-  :ref:`@OneToOne <annref_onetoone>`
-  :ref:`@OneToMany <annref_onetomany>`
-  :ref:`@OrderBy <annref_orderby>`
-  :ref:`@PostLoad <annref_postload>`
-  :ref:`@PostPersist <annref_postpersist>`
-  :ref:`@PostRemove <annref_postremove>`
-  :ref:`@PostUpdate <annref_postupdate>`
-  :ref:`@PrePersist <annref_prepersist>`
-  :ref:`@PreRemove <annref_preremove>`
-  :ref:`@PreUpdate <annref_preupdate>`
-  :ref:`@SequenceGenerator <annref_sequencegenerator>`
-  :ref:`@SqlResultSetMapping <annref_sql_resultset_mapping>`
-  :ref:`@Table <annref_table>`
-  :ref:`@UniqueConstraint <annref_uniqueconstraint>`
-  :ref:`@Version <annref_version>`

参考
---------

.. _annref_column:

@Column
~~~~~~~

将被注释实例的变量标记为“持久”。它必须在实例变量的PHP文档区块注释中。
此变量中的任何值都将作为实例变量的“实体类”的生命周期的一部分保存到数据库中并从数据库中加载。

必需属性：

-  **type**: 在PHP和数据库表示之间转换的Doctrine映射类型的名称。

可选属性：

-  **name**: 默认情况下，属性名称也用于数据库列名称，但 ``name`` 属性允许你修改列名称。

-  **length**: 由 ``string`` 类型使用，用于确定其在数据库中的最大长度。
   Doctrine不会为你验证该字符串值的长度。

-  **precision**: 一个十进制（精确数字）列的精度（仅适用于十进制列），这是该值存储的最大位数。

-  **scale**: 一个十进制（精确数字）列的小数点(scale)（仅适用于十进制列），表示小数点右边的位数，且不得大于 *precision*。

-  **unique**: 布尔值，用于确定列的值在底层实体表的所有行中是否唯一。

-  **nullable**: 确定此列是否允许 ``NULL`` 值。如果未指定，则默认值为 ``false``。

-  **options**: 其他选项数组：

   -  ``default``: 如果未提供任何值，则为列设置的默认值。

   -  ``unsigned``: 布尔值，用于确定列是否应仅能够表示非负整数（仅适用于整数列，并且可能不受所有供应商支持）。

   -  ``fixed``: 布尔值，用于确定字符串列的指定长度是固定还是变化
      （仅适用于字符串/二进制列，并且可能不受所有供应商支持）。

   -  ``comment``: 模式中列的注释（可能并非所有供应商都支持）。

   -  ``collation``: 列的排序规则（仅受Drizzle，Mysql，PostgreSQL> = 9.1，Sqlite和SQLServer支持）。

   -  ``check``: 向列添加一个检查约束类型（可能并非所有供应商都支持）。

-  **columnDefinition**: 在列名后面开始并指定完整（非可移植！）的列定义的DDL SQL片段。
   此属性允许使用高级RMDBS功能。但是，你应该小心使用此功能及其后果。
   如果使用 ``columnDefinition``，``SchemaTool`` 将无法再正确检测列的更改。

   此外，你应该记住 ``type`` 属性仍然处理PHP和数据库值之间的转换。
   如果在用于表之间连接的列上使用此属性，则还应该查看 :ref:`@JoinColumn <annref_joincolumn>`。

.. note::

    有关每个属性的更多详细信息，请参阅DBAL的 ``Schema-Representation`` 文档。

示例：

.. code-block:: php

    /**
     * @Column(type="string", length=32, unique=true, nullable=false)
     */
    protected $username;

    /**
     * @Column(type="string", columnDefinition="CHAR(2) NOT NULL")
     */
    protected $country;

    /**
     * @Column(type="decimal", precision=2, scale=1)
     */
    protected $height;

    /**
     * @Column(type="string", length=2, options={"fixed":true, "comment":"Initial letters of first and last name"})
     */
    protected $initials;

    /**
     * @Column(type="integer", name="login_count" nullable=false, options={"unsigned":true, "default":0})
     */
    protected $loginCount;

.. _annref_column_result:

@ColumnResult
~~~~~~~~~~~~~~

引用SQL查询的 ``SELECT`` 子句中的一个列的名称。
通过在元数据中指定此注释，可以在查询结果中包含 ``Scalar`` 结果类型。

必需属性：

-  **name**: SQL查询的 ``SELECT`` 子句中一个列的名称

.. _annref_cache:

@Cache
~~~~~~~~~~~~~~

将缓存策略添加到一个根实体或集合。

可选属性：

-  **usage**: ``READ_ONLY``、``READ_WRITE`` 或 ``NONSTRICT_READ_WRITE`` 其中之一，默认值为 ``READ_ONLY``。
-  **region**: 特定的区域(region)名称

.. _annref_changetrackingpolicy:

@ChangeTrackingPolicy
~~~~~~~~~~~~~~~~~~~~~

变更跟踪策略注释允许指定Doctrine2的 ``UnitOfWork`` 在刷新期间应如何检测实体的属性的变更。
默认情况下，根据延迟隐式策略将检查每个实体，这意味着在刷新时，``UnitOfWork`` 会将一个实体的所有属性与先前存储的快照进行比较。
这开箱即用，但你可能想要调整刷新的性能，使用其他变更跟踪策略是一个有趣的选择。

有关所有可用变更跟踪策略的详细信息，请参阅 :doc:`配置部分 <change-tracking-policies>`。

示例：

.. code-block:: php

    <?php
    /**
     * @Entity
     * @ChangeTrackingPolicy("DEFERRED_IMPLICIT")
     * @ChangeTrackingPolicy("DEFERRED_EXPLICIT")
     * @ChangeTrackingPolicy("NOTIFY")
     */
    class User {}

.. _annref_customidgenerator:

@CustomIdGenerator
~~~~~~~~~~~~~~~~~~~~~

此注释允许你指定一个自定义的类以生成标识符。
只有在指定 :ref:`@Id <annref_id>` 和 :ref:`@GeneratedValue(strategy="CUSTOM") <annref_generatedvalue>`
时，此注释才有效。

必需属性：

-  **class**: 继承 ``Doctrine\ORM\Id\AbstractIdGenerator`` 类的类名称

示例：

.. code-block:: php

    <?php
    /**
     * @Id
     * @Column(type="integer")
     * @GeneratedValue(strategy="CUSTOM")
     * @CustomIdGenerator(class="My\Namespace\MyIdGenerator")
     */
    public $id;

.. _annref_discriminatorcolumn:

@DiscriminatorColumn
~~~~~~~~~~~~~~~~~~~~~

此注释是一个继承层级的顶层/超类的可选注释。它用于指定保存类实际名称的类的详细信息。
This annotation is an optional annotation for the topmost/super
class of an inheritance hierarchy.
It specifies the details of the column which saves the name of the class, 
which the entity is actually instantiated as.
如果未指定此注释，则 ``discriminator`` 列默认为一个长度 ``255``、名为 ``dtype`` 的字符串列。

必需属性：


-  **name**: 鉴别器的列名称。在数组融合期间，此名称也用作指定类名的键。

可选属性：


-  **type**: 默认为 ``string`` 类型
-  **length**: 默认为 ``255``。

.. _annref_discriminatormap:

@DiscriminatorMap
~~~~~~~~~~~~~~~~~~~~~

鉴别器映射是一个继承层级中顶层/超类的必需注释。
它唯一的参数是一个数组，它定义了应该在数据库中的使用哪个名称来保存对应的类。
数组中的键是数据库的表名称，值是对应的类，该类可以是完全或非限定类名，具体取决于该类是否在命名空间中。

.. code-block:: php

    <?php
    /**
     * @Entity
     * @InheritanceType("JOINED")
     * @DiscriminatorColumn(name="discr", type="string")
     * @DiscriminatorMap({"person" = "Person", "employee" = "Employee"})
     */
    class Person
    {
        // ...
    }


.. _annref_embeddable:

@Embeddable
~~~~~~~~~~~~~~~~~~~~~

要使一个类可被嵌入到一个实体中，则该类必需 ``@Embeddable`` 注释。
该注释与 :ref:`@Embedded <annref_embedded>` 注释一起使用，以建立两个类之间的关系。

.. code-block:: php

    <?php

    /**
     * @Embeddable
     */
    class Address
    {
        // ...
    }
    class User
    {
        /**
         * @Embedded(class = "Address")
         */
        private $address;
    }

.. _annref_embedded:

@Embedded
~~~~~~~~~~~~~~~~~~~~~

要指定一个实体的成员变量是一个嵌入式类，则必须使用 ``@Embedded`` 注释。

必需属性：

-  **class**: 可嵌入的类


.. code-block:: php

    // ...
    class User
    {
        /**
         * @Embedded(class = "Address")
         */
        private $address;
    }

    /**
     * @Embeddable
     */
    class Address
    {
    // ...
    }

.. _annref_entity:

@Entity
~~~~~~~

要将一个PHP类标记为实体，则必须使用 ``@Entity`` 注释。Doctrine管理着所有标记为实体的类的持久性。

可选属性：

-  **repositoryClass**: 指定 ``EntityRepository`` 的子类的FQCN。
   鼓励使用实体的仓库来保持专用DQL/SQL操作与模型/域层的分离。
-  **readOnly**:（> = 2.1）指定此实体标记为只读，不考虑变更跟踪。可以持久化和删除此类型的实体。

示例：

.. code-block:: php

    <?php
    /**
     * @Entity(repositoryClass="MyProject\UserRepository")
     */
    class User
    {
        //...
    }

.. _annref_entity_result:

@EntityResult
~~~~~~~~~~~~~~

引用SQL查询的 ``SELECT`` 子句中的一个实体。如果使用此注释，则SQL语句应选择映射到该实体对象的所有列。
这应该包括相关实体的外键列。当数据不足时获得的结果是未定义的。

必需属性：

-  **entityClass**: 结果的类。

可选属性：

-  **fields**: Array of @FieldResult, Maps the columns specified in the SELECT list of the query to the properties or fields of the entity class.``@FieldResult`` 数组，将查询的 ``SELECT`` 列表中指定的列映射到实体类的属性或字段。
-  **discriminatorColumn**: Specifies the column name of the column in the SELECT list that is used to determine the type of the entity instance.指定 ``SELECT`` 列表中用于确定实体实例的类型的列的列名。

.. _annref_field_result:

@FieldResult
~~~~~~~~~~~~~

用于将查询的 ``SELECT`` 列表中指定的列映射到实体类的属性或字段。

必需属性：

-  **name**: 类的持久字段或属性的名称。


可选属性：

-  **column**: ``SELECT`` 子句中列的名称。

.. _annref_generatedvalue:

@GeneratedValue
~~~~~~~~~~~~~~~~~~~~~

为一个用 :ref:`@Id <annref_id>` 注释的实例变量指定生成标识符的策略，
此注释是可选的，仅在与 :ref:`@Id <annref_id>` 一起使用时才有意义。

如果未使用 ``@Id`` 指定此注释，则将使用 ``NONE`` 策略作为默认策略。

可选属性：

-  **strategy**: 设置标识符生成策略的名称。
   有效值为``AUTO``、``SEQUENCE``、``TABLE``、``IDENTITY``、``UUID``、``CUSTOM`` 以及 ``NONE``。
   如果未指定，则默认值为 ``AUTO``。

示例：

.. code-block:: php

    <?php
    /**
     * @Id
     * @Column(type="integer")
     * @GeneratedValue(strategy="IDENTITY")
     */
    protected $id = null;

.. _annref_haslifecyclecallbacks:

@HasLifecycleCallbacks
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

必须在实体类的PHP文档区块上设置此注释，以通知Doctrine该实体至少设置了一个实体生命周期回调注释。
使用了 ``@PostLoad``、``@PrePersist``、``@PostPersist``、``@PreRemove``、``@PostRemove``、``@PreUpdate`` 或
``@PostUpdate`` 注释，但没有使用此注释，Doctrine将会忽略该回调。

示例：

.. code-block:: php

    <?php
    /**
     * @Entity
     * @HasLifecycleCallbacks
     */
    class User
    {
        /**
         * @PostPersist
         */
        public function sendOptinMail() {}
    }

.. _annref_index:

@Index
~~~~~~~

此注释在实体级别的 :ref:`@Table <annref_table>` 注释中使用。
它为 ``SchemaTool`` 提供了一个提示，以便在指定表的列上生成一个数据库索引。
它仅在 ``SchemaTool`` 模式生成的上下文中有意义。

必需属性：


-  **name**: 索引的名称
-  **columns**: 列数组。

可选属性：

-  **options**: 特定于平台的选项数组：

   -  ``where``: 用于部分索引的SQL ``WHERE`` 条件。它只会对受支持的平台产生影响。

基本示例：

.. code-block:: php

    /**
     * @Entity
     * @Table(name="ecommerce_products",indexes={@Index(name="search_idx", columns={"name", "email"})})
     */
    class ECommerceProduct
    {
    }

部分索引的示例：

.. code-block:: php

    <?php
    /**
     * @Entity
     * @Table(name="ecommerce_products",indexes={@Index(name="search_idx", columns={"name", "email"}, options={"where": "(((id IS NOT NULL) AND (name IS NULL)) AND (email IS NULL))"})})
     */
    class ECommerceProduct
    {
    }

.. _annref_id:

@Id
~~~~~~~

带此注释的实例变量将被标记为实体的标识符，即数据库中的主键。
此注释仅是一个标记，没有必需或可选属性。对于具有多个标识符列的实体，每个列都必须使用 ``@Id`` 进行标记。

示例：

.. code-block:: php

    <?php
    /**
     * @Id
     * @Column(type="integer")
     */
    protected $id = null;

.. _annref_inheritancetype:

@InheritanceType
~~~~~~~~~~~~~~~~~~~~~

在一个继承层次中，你必须在最顶层/超类上使用此注释来定义应该用于继承的策略。目前支持单表和类表继承。

此注释始终与 :ref:`@DiscriminatorMap <annref_discriminatormap>` 和
:ref:`@DiscriminatorColumn <annref_discriminatorcolumn>` 注释一起使用。

示例：

.. code-block:: php

    <?php
    /**
     * @Entity
     * @InheritanceType("SINGLE_TABLE")
     * @DiscriminatorColumn(name="discr", type="string")
     * @DiscriminatorMap({"person" = "Person", "employee" = "Employee"})
     */
    class Person
    {
        // ...
    }

    /**
     * @Entity
     * @InheritanceType("JOINED")
     * @DiscriminatorColumn(name="discr", type="string")
     * @DiscriminatorMap({"person" = "Person", "employee" = "Employee"})
     */
    class Person
    {
        // ...
    }

.. _annref_joincolumn:

@JoinColumn
~~~~~~~~~~~~~~

此注释用于 :ref:`@ManyToOne <annref_manytoone>`、:ref:`@OneToOne <annref_onetoone>`
字段的关系上下文中，以及嵌套在 :ref:`@ManyToMany <annref_manytomany>` 内的
:ref:`@JoinTable <annref_jointable>` 的上下文中。
此注释不是必需的。如果未指定，则从表和主键的名称中推断出该属性的 ``name`` 和 ``referencedColumnName``。

必需属性：

-  **name**: 保持此关系的外键标识符的列名。在 ``@JoinTable`` 的上下文中，它指定了连接表中的列名。
-  **referencedColumnName**: 用于连接此关系的主键标识符的名称。

可选属性：

-  **unique**: 确定此关系在受影响的实体之间是否是唯一的，并且应在数据库约束级别上强制执行。默认为 ``false``。
-  **nullable**: 确定相关实体是否是必需的，或者 ``null`` 是否为此关系的允许状态。默认为 ``true``。
-  **onDelete**: 级联动作（数据库级）
-  **columnDefinition**: 在列名后面开始并指定完整（非可移植！）列定义的DDL SQL片段。
   此属性允许使用高级RMDBS功能。如果你需要稍微不同的列定义来连接列，则必须在
   ``@JoinColumn`` 上使用此属性，例如，关于 ``NULL`` / ``NOT NULL`` 默认值。
   但是，默认情况下，:ref:`@Column <annref_column>` 上的一个 ``columnDefinition``
   属性也会设置相关的 ``@JoinColumn`` 的 ``columnDefinition``。这是使外键生效所必需的配置。

示例：

.. code-block:: php

    <?php
    /**
     * @OneToOne(targetEntity="Customer")
     * @JoinColumn(name="customer_id", referencedColumnName="id")
     */
    private $customer;

.. _annref_joincolumns:

@JoinColumns
~~~~~~~~~~~~~~

与具有多个标识符的一个实体的 :ref:`@ManyToOne <annref_manytoone>` 或
:ref:`@OneToOne <annref_onetoone>` 关系的
:ref:`@JoinColumn <annref_joincolumn>` 注释数组。

.. _annref_jointable:

@JoinTable
~~~~~~~~~~~~~~

在关系的拥有方使用 :ref:`@OneToMany <annref_onetomany>` 或
:ref:`@ManyToMany <annref_manytomany>` 时需要指定 ``@JoinTable``
注释，该注释用于描述数据库连接表的详细信息。
如果未在这些关系上指定 ``@JoinTable``，则使用受影响的表和列名称来应用合理的映射默认值。

可选属性：

-  **name**: 连接表的名称
-  **joinColumns**: 一个 ``@JoinColumn`` 注释数组，用于描述拥有方实体表和连接表之间的连接关系。
-  **inverseJoinColumns**: 一个 ``@JoinColumn`` 注释数组，用于描述方从属方实体表和连接表之间的连接关系。

示例：

.. code-block:: php

    <?php
    /**
     * @ManyToMany(targetEntity="Phonenumber")
     * @JoinTable(name="users_phonenumbers",
     *      joinColumns={@JoinColumn(name="user_id", referencedColumnName="id")},
     *      inverseJoinColumns={@JoinColumn(name="phonenumber_id", referencedColumnName="id", unique=true)}
     * )
     */
    public $phonenumbers;

.. _annref_manytoone:

@ManyToOne
~~~~~~~~~~~~~~

定义被注释的实例变量持有一个描述两个实体之间多对一关系的引用。

必需属性：

-  **targetEntity**: 引用的目标实体的FQCN。
   如果两个类都在同一命名空间中，则可以是非限定类名。*重要提示*：没有领头的反斜杠！

可选属性：


-  **cascade**: 级联选项
-  **fetch**: ``LAZY`` 或 ``EAGER`` 之一
-  **inversedBy**: - 该属性用于指定此关系的从属方实体中的字段。

示例：

.. code-block:: php

    <?php
    /**
     * @ManyToOne(targetEntity="Cart", cascade={"all"}, fetch="EAGER")
     */
    private $cart;

.. _annref_manytomany:

@ManyToMany
~~~~~~~~~~~~~~

定义被注释的实例变量持有一个描述两个实体之间多对多关系的引用。
:ref:`@JoinTable <annref_jointable>` 是一个附加、可选的注释，它使用两个相关实体的表和名称来配置合理的默认值。

必需属性：

-  **targetEntity**: 引用的目标实体的FQCN。
   如果两个类都在同一命名空间中，则可以是非限定类名。*重要提示*：没有领头的反斜杠！

可选属性：

-  **mappedBy**: 此选项指定 ``targetEntity`` (此关系的拥有方)上的属性名称。它是从属方关系的必需属性。
-  **inversedBy**: 该属性用于指定此关系的从属方实体中的字段。
-  **cascade**: 级联选项
-  **fetch**: ``LAZY``、``EXTRA_LAZY`` 或 ``EAGER`` 之一
-  **indexBy**: 通过目标实体上的字段来索引集合。

.. note::

    For ManyToMany bidirectional relationships either side may
    be the owning side (the side that defines the @JoinTable and/or
    does not make use of the mappedBy attribute, thus using a default
    join table).
    对于双向的 ``ManyToMany`` 关系，任何一方都可以是拥有方（定义
    ``@JoinTable`` 的一方 *和/或* 不使用 ``mappedBy`` 属性，因此使用默认连接表）。

示例：

.. code-block:: php

    <?php
    /**
     * Owning Side
     *
     * @ManyToMany(targetEntity="Group", inversedBy="features")
     * @JoinTable(name="user_groups",
     *      joinColumns={@JoinColumn(name="user_id", referencedColumnName="id")},
     *      inverseJoinColumns={@JoinColumn(name="group_id", referencedColumnName="id")}
     *      )
     */
    private $groups;

    /**
     * Inverse Side
     *
     * @ManyToMany(targetEntity="User", mappedBy="groups")
     */
    private $features;

.. _annref_mappedsuperclass:

@MappedSuperclass
~~~~~~~~~~~~~~~~~~~~~

被映射的超类是一个抽象或具体的类，它为其子类提供持久的实体状态和映射信息，但它本身不是一个实体。
此注释在类的文档区块上指定，并且没有其他属性。

``@MappedSuperclass`` 注释不能与 ``@Entity`` 一起使用。
有关映射超类限制的更多详细信息，请参阅 :doc:`继承映射 <inheritance-mapping>` 章节。

可选属性：

-  **repositoryClass**:（> = 2.2）指定 ``EntityRepository`` 的子类的FQCN。
   这将继承该已映射超类的所有子类。

示例：

.. code-block:: php

    <?php
    /**
     * @MappedSuperclass
     */
    class MappedSuperclassBase
    {
        // ... 字段和方法
    }

    /**
     * @Entity
     */
    class EntitySubClassFoo extends MappedSuperclassBase
    {
        // ... 字段和方法
    }

.. _annref_named_native_query:

@NamedNativeQuery
~~~~~~~~~~~~~~~~~

用于指定一个原生SQL命名查询。此注释可以应用于实体或已映射超类。

必需属性：

-  **name**: The name used to refer to the query with the EntityManager methods that create query objects.用于引用通过创建查询对象的EntityManager方法的查询的名称。
-  **query**: SQL查询字符串

可选属性：

-  **resultClass**: 结果的类。
-  **resultSetMapping**: 一个在元数据中定义的 ``SqlResultSetMapping`` 的名称。

示例：

.. code-block:: php

    <?php
    /**
     * @NamedNativeQueries({
     *      @NamedNativeQuery(
     *          name            = "fetchJoinedAddress",
     *          resultSetMapping= "mappingJoinedAddress",
     *          query           = "SELECT u.id, u.name, u.status, a.id AS a_id, a.country, a.zip, a.city FROM cms_users u INNER JOIN cms_addresses a ON u.id = a.user_id WHERE u.username = ?"
     *      ),
     * })
     * @SqlResultSetMappings({
     *      @SqlResultSetMapping(
     *          name    = "mappingJoinedAddress",
     *          entities= {
     *              @EntityResult(
     *                  entityClass = "__CLASS__",
     *                  fields      = {
     *                      @FieldResult(name = "id"),
     *                      @FieldResult(name = "name"),
     *                      @FieldResult(name = "status"),
     *                      @FieldResult(name = "address.zip"),
     *                      @FieldResult(name = "address.city"),
     *                      @FieldResult(name = "address.country"),
     *                      @FieldResult(name = "address.id", column = "a_id"),
     *                  }
     *              )
     *          }
     *      )
     * })
     */
    class User
    {
        /** @Id @Column(type="integer") @GeneratedValue */
        public $id;

        /** @Column(type="string", length=50, nullable=true) */
        public $status;

        /** @Column(type="string", length=255, unique=true) */
        public $username;

        /** @Column(type="string", length=255) */
        public $name;

        /** @OneToOne(targetEntity="Address") */
        public $address;

        // ....
    }
.. _annref_onetoone:

@OneToOne
~~~~~~~~~~~~~~

``@OneToOne`` 注释几乎与 :ref:`@ManyToOne <annref_manytoone>` 一样，除了只能指定一个附加选项。
使用目标实体的表和主键列名的 ``@JoinColumn`` 的配置默认值也适用于此处。

必需属性：

-  **targetEntity**: 引用的目标实体的FQCN。
   如果两个类都在同一命名空间中，则可以是非限定类名。*重要提示*：没有领头的反斜杠！

可选属性：


-  **cascade**: 级联选项
-  **fetch**: ``LAZY`` 或 ``EAGER`` 之一
-  **orphanRemoval**: 布尔值，用于指定是否应该由Doctrine删除孤立(orphan)，孤立是指未连接到任何拥有方实例的从属方一对一实体。默认为 ``false``。
-  **inversedBy**: 该属性用于指定此关系的从属方实体中的字段。

示例：

.. code-block:: php

    <?php
    /**
     * @OneToOne(targetEntity="Customer")
     * @JoinColumn(name="customer_id", referencedColumnName="id")
     */
    private $customer;

.. _annref_onetomany:

@OneToMany
~~~~~~~~~~~~~~

必需属性：

-  **targetEntity**: 引用的目标实体的FQCN。
   如果两个类都在同一命名空间中，则可以是非限定类名。*重要提示*：没有领头的反斜杠！

可选属性：

-  **cascade**: 级联选项
-  **orphanRemoval**: 布尔值，用于指定是否应该由Doctrine删除孤立(orphan)，孤立是指未连接到任何拥有方实例的从属方一对一实体。默认为 ``false``。
-  **mappedBy**: 此选项指定 ``targetEntity`` (此关系的拥有方)上的属性名称。它是从属方关系的必需属性。
-  **fetch**: ``LAZY``、``EXTRA_LAZY`` 或 ``EAGER`` 之一。
-  **indexBy**: 通过目标实体上的字段来索引集合。

示例：

.. code-block:: php

    <?php
    /**
     * @OneToMany(targetEntity="Phonenumber", mappedBy="user", cascade={"persist", "remove", "merge"}, orphanRemoval=true)
     */
    public $phonenumbers;

.. _annref_orderby:

@OrderBy
~~~~~~~~~~~~~~

可以在使用 :ref:`@ManyToMany <annref_manytomany>` 或 :ref:`@OneToMany <annref_onetomany>` 的注释上指定的可选注释，以指定应使用 ``ORDER BY`` 子句从数据库中检索集合的条件。

此注释要求单个使用DQL代码段的非属性值：

示例：

.. code-block:: php

    /**
     * @ManyToMany(targetEntity="Group")
     * @OrderBy({"name" = "ASC"})
     */
    private $groups;

``@OrderBy`` 中的DQL代码段只允许由未限定的、未引用的字段名称和一个可选的
``ASC`` / ``DESC`` 位置语句组成。多个字段可用逗号（``,``）分隔。
引用的字段名称必须存在于 ``@ManyToMany`` 或 ``@OneToMany`` 注释的 ``targetEntity`` 类中。

.. _annref_postload:

@PostLoad
~~~~~~~~~~~~~~

标记实体上的一个方法，该方法将被 ``@PostLoad`` 事件调用。
此注释仅限于与实体类的PHP文档区块中的 ``@HasLifecycleCallbacks`` 注释一起使用。

.. _annref_postpersist:

@PostPersist
~~~~~~~~~~~~~~

标记实体上的一个方法，该方法将被 ``@PostPersist`` 事件调用。
此注释仅限于与实体类的PHP文档区块中的 ``@HasLifecycleCallbacks`` 注释一起使用。

.. _annref_postremove:

@PostRemove
~~~~~~~~~~~~~~

标记实体上的一个方法，该方法将被 ``@PostRemove`` 事件调用。
此注释仅限于与实体类的PHP文档区块中的 ``@HasLifecycleCallbacks`` 注释一起使用。

.. _annref_postupdate:

@PostUpdate
~~~~~~~~~~~~~~

标记实体上的一个方法，该方法将被 ``@PostUpdate`` 事件调用。
此注释仅限于与实体类的PHP文档区块中的 ``@HasLifecycleCallbacks`` 注释一起使用。

.. _annref_prepersist:

@PrePersist
~~~~~~~~~~~~~~

标记实体上的一个方法，该方法将被 ``@PrePersist`` 事件调用。
此注释仅限于与实体类的PHP文档区块中的 ``@HasLifecycleCallbacks`` 注释一起使用。

.. _annref_preremove:

@PreRemove
~~~~~~~~~~~~~~

标记实体上的一个方法，该方法将被 ``@PreRemove`` 事件调用。
此注释仅限于与实体类的PHP文档区块中的 ``@HasLifecycleCallbacks`` 注释一起使用。

.. _annref_preupdate:

@PreUpdate
~~~~~~~~~~~~~~

标记实体上的一个方法，该方法将被 ``@PreUpdate`` 事件调用。
此注释仅限于与实体类的PHP文档区块中的 ``@HasLifecycleCallbacks`` 注释一起使用。

.. _annref_sequencegenerator:

@SequenceGenerator
~~~~~~~~~~~~~~~~~~~~~

此注释与 ``@GeneratedValue(strategy="SEQUENCE")``
配套使用，它允许指定关于序列的细节，例如序列的增量大小和初始值。

必需属性：

-  **sequenceName**: 序列的名称

可选属性：

-  **allocationSize**: 在获取序列时，按照分配大小递增该序列。
   大于 ``1`` 的值允许对每个请求创建多个新实体的情况进行优化。默认为 ``10``
-  **initialValue**: 指定序列应从哪个值开始，默认为 ``1``。

示例：

.. code-block:: php

    /**
     * @Id
     * @GeneratedValue(strategy="SEQUENCE")
     * @Column(type="integer")
     * @SequenceGenerator(sequenceName="tablename_seq", initialValue=1, allocationSize=100)
     */
    protected $id = null;

.. _annref_sql_resultset_mapping:

@SqlResultSetMapping
~~~~~~~~~~~~~~~~~~~~

此注释用于指定一个原生SQL查询结果的映射。此注释可以应用于一个实体或已映射超类。

必需属性：

-  **name**: 给结果集映射的名称，用于在Query API的方法中引用它。

可选属性：

-  **entities**: ``@EntityResult`` 数组，指定映射到实体的结果集。
-  **columns**: ``@ColumnResult`` 数组，指定映射到标量值的结果集。

示例：

.. code-block:: php

    <?php
    /**
     * @NamedNativeQueries({
     *      @NamedNativeQuery(
     *          name            = "fetchUserPhonenumberCount",
     *          resultSetMapping= "mappingUserPhonenumberCount",
     *          query           = "SELECT id, name, status, COUNT(phonenumber) AS numphones FROM cms_users INNER JOIN cms_phonenumbers ON id = user_id WHERE username IN (?) GROUP BY id, name, status, username ORDER BY username"
     *      ),
     *      @NamedNativeQuery(
     *          name            = "fetchMultipleJoinsEntityResults",
     *          resultSetMapping= "mappingMultipleJoinsEntityResults",
     *          query           = "SELECT u.id AS u_id, u.name AS u_name, u.status AS u_status, a.id AS a_id, a.zip AS a_zip, a.country AS a_country, COUNT(p.phonenumber) AS numphones FROM cms_users u INNER JOIN cms_addresses a ON u.id = a.user_id INNER JOIN cms_phonenumbers p ON u.id = p.user_id GROUP BY u.id, u.name, u.status, u.username, a.id, a.zip, a.country ORDER BY u.username"
     *      ),
     * })
     * @SqlResultSetMappings({
     *      @SqlResultSetMapping(
     *          name    = "mappingUserPhonenumberCount",
     *          entities= {
     *              @EntityResult(
     *                  entityClass = "User",
     *                  fields      = {
     *                      @FieldResult(name = "id"),
     *                      @FieldResult(name = "name"),
     *                      @FieldResult(name = "status"),
     *                  }
     *              )
     *          },
     *          columns = {
     *              @ColumnResult("numphones")
     *          }
     *      ),
     *      @SqlResultSetMapping(
     *          name    = "mappingMultipleJoinsEntityResults",
     *          entities= {
     *              @EntityResult(
     *                  entityClass = "__CLASS__",
     *                  fields      = {
     *                      @FieldResult(name = "id",       column="u_id"),
     *                      @FieldResult(name = "name",     column="u_name"),
     *                      @FieldResult(name = "status",   column="u_status"),
     *                  }
     *              ),
     *              @EntityResult(
     *                  entityClass = "Address",
     *                  fields      = {
     *                      @FieldResult(name = "id",       column="a_id"),
     *                      @FieldResult(name = "zip",      column="a_zip"),
     *                      @FieldResult(name = "country",  column="a_country"),
     *                  }
     *              )
     *          },
     *          columns = {
     *              @ColumnResult("numphones")
     *          }
     *      )
     *})
     */
     class User
    {
        /** @Id @Column(type="integer") @GeneratedValue */
        public $id;

        /** @Column(type="string", length=50, nullable=true) */
        public $status;

        /** @Column(type="string", length=255, unique=true) */
        public $username;

        /** @Column(type="string", length=255) */
        public $name;

        /** @OneToMany(targetEntity="Phonenumber") */
        public $phonenumbers;

        /** @OneToOne(targetEntity="Address") */
        public $address;

        // ....
    }
.. _annref_table:

@Table
~~~~~~~

此注释描述了一个实体持久化的表。它放在实体类的PHP文档区块上并且是可选的。
如果未指定，则表名称将默认为实体的非限定类名。

必需属性：


-  **name**: 表名称

可选属性：


-  **indexes**: :ref:`@Index <annref_index>` 注释数组
-  **uniqueConstraints**: :ref:`@UniqueConstraint <annref_uniqueconstraint>` 注释数组
-  **schema**: （> = 2.5）表所在的模式的名称

示例：

.. code-block:: php

    <?php
    /**
     * @Entity
     * @Table(name="user",
     *      uniqueConstraints={@UniqueConstraint(name="user_unique",columns={"username"})},
     *      indexes={@Index(name="user_idx", columns={"email"})}
     *      schema="schema_name"
     * )
     */
    class User { }

.. _annref_uniqueconstraint:

@UniqueConstraint
~~~~~~~~~~~~~~~~~~~~~

此注释在实体级别的 :ref:`@Table <annref_table>` 注释中使用。
它允许提示 ``SchemaTool`` 在指定的表的列上生成一个数据库唯一约束。
它仅在 ``SchemaTool`` 模式生成上下文中有意义。

必需属性：


-  **name**: 索引的名称
-  **columns**: 列数组。

可选属性：

-  **options**: 特定于平台的选项数组：

   -  ``where``: 用于部分索引的SQL ``WHERE`` 条件。它只会对支持的平台产生影响。

基本示例：

.. code-block:: php

    <?php
    /**
     * @Entity
     * @Table(name="ecommerce_products",uniqueConstraints={@UniqueConstraint(name="search_idx", columns={"name", "email"})})
     */
    class ECommerceProduct
    {
    }

部分索引的示例：

.. code-block:: php

    <?php
    /**
     * @Entity
     * @Table(name="ecommerce_products",uniqueConstraints={@UniqueConstraint(name="search_idx", columns={"name", "email"}, options={"where": "(((id IS NOT NULL) AND (name IS NULL)) AND (email IS NULL))"})})
     */
    class ECommerceProduct
    {
    }

.. _annref_version:

@Version
~~~~~~~~

标记注释，将一个指定列定义为一个 :ref:`乐观锁 <transactions-and-concurrency_optimistic-locking>`
方案中使用的版本属性。它仅适用于具有 ``integer`` 或 ``datetime``
类型的 :ref:`@Column <annref_column>` 注释。不支持与 :ref:`@Id <annref_id>` 结合使用。

示例：

.. code-block:: php

    <?php
    /**
     * @Column(type="integer")
     * @Version
     */
    protected $version;
