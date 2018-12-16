继承映射
===================

映射超类
-------------------

一个已映射超类是一个抽象或具体的类，它为其子类提供持久的实体状态和映射信息，但它本身不是一个实体。
通常，这种已映射超类的目的是定义多个实体类共有的状态和映射信息。
A mapped superclass is an abstract or concrete class that provides
persistent entity state and mapping information for its subclasses,
but which is not itself an entity. Typically, the purpose of such a
mapped superclass is to define state and mapping information that
is common to multiple entity classes.

已映射超类与常规的非映射类一样，可以出现在其他映射的继承层级的中间（通过单表继承或类表继承）。
Mapped superclasses, just as regular, non-mapped classes, can
appear in the middle of an otherwise mapped inheritance hierarchy
(through Single Table Inheritance or Class Table Inheritance).

.. note::

    A mapped superclass cannot be an entity, it is not query-able and
    persistent relationships defined by a mapped superclass must be
    unidirectional (with an owning side only). This means that One-To-Many
    associations are not possible on a mapped superclass at all.
    Furthermore Many-To-Many associations are only possible if the
    mapped superclass is only used in exactly one entity at the moment.
    For further support of inheritance, the single or
    joined table inheritance features have to be used.
    已映射超类不能是实体，它不是可查询的，并且由已映射超类定义的持久关系必须是单向的（仅具有拥有方）。
    这意味着根本不可能在已映射超类上进行一对多关联。
    此外，只有当已映射超类目前仅在一个实体中使用时，才能实现多对多关联。
    为了进一步支持继承，必须使用单个或连接表的继承功能。

示例：

.. code-block:: php

    <?php
    /** @MappedSuperclass */
    class MappedSuperclassBase
    {
        /** @Column(type="integer") */
        protected $mapped1;
        /** @Column(type="string") */
        protected $mapped2;
        /**
         * @OneToOne(targetEntity="MappedSuperclassRelated1")
         * @JoinColumn(name="related1_id", referencedColumnName="id")
         */
        protected $mappedRelated1;

        // ... more fields and methods
    }

    /** @Entity */
    class EntitySubClass extends MappedSuperclassBase
    {
        /** @Id @Column(type="integer") */
        private $id;
        /** @Column(type="string") */
        private $name;

        // ... more fields and methods
    }

相应数据库模式的DDL看起来像这样（这适用于SQLite）：
The DDL for the corresponding database schema would look something
like this (this is for SQLite):

.. code-block:: sql

    CREATE TABLE EntitySubClass (mapped1 INTEGER NOT NULL, mapped2 TEXT NOT NULL, id INTEGER NOT NULL, name TEXT NOT NULL, related1_id INTEGER DEFAULT NULL, PRIMARY KEY(id))

从这个DDL片段可以看出，实体子类只有一个表。映射的超类中的所有映射都继承到子类，就好像它们已直接在该类上定义一样。
As you can see from this DDL snippet, there is only a single table
for the entity subclass. All the mappings from the mapped
superclass were inherited to the subclass as if they had been
defined on that class directly.

单表继承
------------------------

单表继承 是一种继承映射策略，其中层次结构的所有类都映射到单个数据库表。为了区分哪一行表示层次结构中的哪种类型，使用所谓的鉴别器列。
`Single Table Inheritance <http://martinfowler.com/eaaCatalog/singleTableInheritance.html>`_
is an inheritance mapping strategy where all classes of a hierarchy
are mapped to a single database table. In order to distinguish
which row represents which type in the hierarchy a so-called
discriminator column is used.

示例：

.. configuration-block::

    .. code-block:: php

        <?php
        namespace MyProject\Model;

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
         */
        class Employee extends Person
        {
            // ...
        }

    .. code-block:: yaml

        MyProject\Model\Person:
          type: entity
          inheritanceType: SINGLE_TABLE
          discriminatorColumn:
            name: discr
            type: string
          discriminatorMap:
            person: Person
            employee: Employee

        MyProject\Model\Employee:
          type: entity

注意事项：

-  The @InheritanceType and @DiscriminatorColumn must be specified
   on the topmost class that is part of the mapped entity hierarchy.
   必须在作为映射实体层次结构一部分的最顶层类上指定@InheritanceType和@DiscriminatorColumn。
-  The @DiscriminatorMap specifies which values of the
   discriminator column identify a row as being of a certain type. In
   the case above a value of "person" identifies a row as being of
   type ``Person`` and "employee" identifies a row as being of type
   ``Employee``.
   @DiscriminatorMap指定鉴别器列的哪些值将行标识为某种类型。在上面的情况下，值“person”将行标识为类型Person，“employee”将行标识为类型 Employee。
-  All entity classes that is part of the mapped entity hierarchy
   (including the topmost class) should be specified in the
   @DiscriminatorMap. In the case above Person class included.
   应在@DiscriminatorMap中指定属于映射实体层次结构（包括最顶层类）的所有实体类。在上面的情况下包括Person类。
-  The names of the classes in the discriminator map do not need to
   be fully qualified if the classes are contained in the same
   namespace as the entity class on which the discriminator map is
   applied.
   如果类包含在与应用鉴别器映射的实体类相同的命名空间中，则鉴别器映射中的类的名称不需要完全限定。
-  If no discriminator map is provided, then the map is generated
   automatically. The automatically generated discriminator map
   contains the lowercase short name of each class as key.
   如果未提供鉴别器映射，则自动生成映射。自动生成的鉴别器映射包含每个类的小写短名称作为键。

Design-time considerations
设计时考虑因素
~~~~~~~~~~~~~~~~~~~~~~~~~~

当类型层次结构非常简单和稳定时，这种映射方法很有效。向层次结构添加新类型并向现有超类型添加字段只涉及向表中添加新列，但在大型部署中，这可能会对数据库内的索引和列布局产生负面影响。
This mapping approach works well when the type hierarchy is fairly
simple and stable. Adding a new type to the hierarchy and adding
fields to existing supertypes simply involves adding new columns to
the table, though in large deployments this may have an adverse
impact on the index and column layout inside the database.

性能影响
~~~~~~~~~~~~~~~~~~

此策略非常有效地查询层次结构中的所有类型或特定类型。不需要表连接，只有列出类型标识符的WHERE子句。特别是，涉及采用此映射策略的类型的关系非常有效。
This strategy is very efficient for querying across all types in
the hierarchy or for specific types. No table joins are required,
only a WHERE clause listing the type identifiers. In particular,
relationships involving types that employ this mapping strategy are
very performing.

单表继承存在一般性能考虑：如果多对一或一对一关联的目标实体是STI实体，则出于性能原因，它最好是继承中的叶实体 层次结构，（即没有子类）。 否则Doctrine *不能*创建该实体的代理实例，并且*总是*急切地加载该实体。
There is a general performance consideration with Single Table Inheritance: If the target-entity of a many-to-one or one-to-one association is an STI entity, it is preferable for performance reasons that it be a leaf entity in the inheritance hierarchy, (ie. have no subclasses). Otherwise Doctrine *CANNOT* create proxy instances of this entity and will *ALWAYS* load the entity eagerly.

SQL Schema considerations
SQL架构注意事项
~~~~~~~~~~~~~~~~~~~~~~~~~

要使单表继承在你使用旧数据库模式或自编写数据库模式的情况下工作，你必须确保所有不在根实体中但在任何不同子实体中的列 必须允许空值。 具有NOT NULL约束的列必须位于单表继承层次结构的根实体上。
For Single-Table-Inheritance to work in scenarios where you are using either a legacy database schema or a self-written database schema you have to make sure that all columns that are not in the root entity but in any of the different sub-entities has to allows null values. Columns that have NOT NULL constraints have to be on the root entity of the single-table inheritance hierarchy.

类表继承
-----------------------

类表继承 是一种继承映射策略，其中层次结构中的每个类都映射到多个表：它自己的表和所有父类的表。子类的表通过外键约束链接到父类的表。Doctrine 2通过在层次结构的最顶层表中使用discriminator列来实现此策略，因为这是使用Class Table Inheritance实现多态查询的最简单方法。
`Class Table Inheritance <http://martinfowler.com/eaaCatalog/classTableInheritance.html>`_
is an inheritance mapping strategy where each class in a hierarchy
is mapped to several tables: its own table and the tables of all
parent classes. The table of a child class is linked to the table
of a parent class through a foreign key constraint. Doctrine 2
implements this strategy through the use of a discriminator column
in the topmost table of the hierarchy because this is the easiest
way to achieve polymorphic queries with Class Table Inheritance.

示例：

.. code-block:: php

    <?php
    namespace MyProject\Model;

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

    /** @Entity */
    class Employee extends Person
    {
        // ...
    }

Things to note:


-  The @InheritanceType, @DiscriminatorColumn and @DiscriminatorMap
   must be specified on the topmost class that is part of the mapped
   entity hierarchy.
   必须在作为映射实体层次结构一部分的最顶层类上指定@InheritanceType，@ DisciminatorColumn和@DiscriminatorMap。
-  The @DiscriminatorMap specifies which values of the
   discriminator column identify a row as being of which type. In the
   case above a value of "person" identifies a row as being of type
   ``Person`` and "employee" identifies a row as being of type
   ``Employee``.
   @DiscriminatorMap指定鉴别器列的哪些值将行标识为哪种类型。在上面的情况下，值“person”将行标识为类型 Person，“employee”将行标识为类型 Employee。
-  The names of the classes in the discriminator map do not need to
   be fully qualified if the classes are contained in the same
   namespace as the entity class on which the discriminator map is
   applied.
   如果类包含在与应用鉴别器映射的实体类相同的命名空间中，则鉴别器映射中的类的名称不需要完全限定。
-  If no discriminator map is provided, then the map is generated automatically. The automatically generated discriminator map contains the lowercase short name of each class as key.
   如果未提供鉴别器映射，则自动生成映射。 自动生成的鉴别器映射包含每个类的小写短名称作为键。

.. note::

    When you do not use the SchemaTool to generate the
    required SQL you should know that deleting a class table
    inheritance makes use of the foreign key property
    ``ON DELETE CASCADE`` in all database implementations. A failure to
    implement this yourself will lead to dead rows in the database.
    当你不使用SchemaTool生成所需的SQL时，你应该知道删除类表继承会ON DELETE CASCADE在所有数据库实现中使用外键属性 。如果未能自行实现，将导致数据库中出现死行。

Design-time considerations
设计时考虑因素
~~~~~~~~~~~~~~~~~~~~~~~~~~

在任何级别向层次结构引入新类型只需将新表插入到模式中。该类型的子类型将在运行时自动与该新类型连接。同样，通过添加，修改或删除字段来修改层次结构中的任何实体类型只会影响映射到该类型的直接表。此映射策略在设计时提供了最大的灵活性，因为对任何类型的更改始终仅限于该类型的专用表。
Introducing a new type to the hierarchy, at any level, simply
involves interjecting a new table into the schema. Subtypes of that
type will automatically join with that new type at runtime.
Similarly, modifying any entity type in the hierarchy by adding,
modifying or removing fields affects only the immediate table
mapped to that type. This mapping strategy provides the greatest
flexibility at design time, since changes to any type are always
limited to that type's dedicated table.

性能影响
~~~~~~~~~~~~~~~~~~

此策略本身需要多个JOIN操作才能执行任何可能对性能产生负面影响的查询，尤其是对于大型表和/或大型层次结构。当全局或特定查询允许部分对象时，查询任何类型都不会导致子类型表为OUTER JOINed，这可以提高性能但是生成的部分对象在访问任何子类型字段时都不会完全加载，所以在这样的查询之后访问子类型的字段是不安全的。
This strategy inherently requires multiple JOIN operations to
perform just about any query which can have a negative impact on
performance, especially with large tables and/or large hierarchies.
When partial objects are allowed, either globally or on the
specific query, then querying for any type will not cause the
tables of subtypes to be OUTER JOINed which can increase
performance but the resulting partial objects will not fully load
themselves on access of any subtype fields, so accessing fields of
subtypes after such a query is not safe.

类表继承存在一般性能考虑因素：如果多对一或一对一关联的目标实体是CTI实体，则出于性能原因，它最好是继承中的叶实体 层次结构，（即没有子类）。 否则Doctrine *不能*创建该实体的代理实例，并且*总是*急切地加载该实体。
There is a general performance consideration with Class Table Inheritance: If the target-entity of a many-to-one or one-to-one association is a CTI entity, it is preferable for performance reasons that it be a leaf entity in the inheritance hierarchy, (ie. have no subclasses). Otherwise Doctrine *CANNOT* create proxy instances of this entity and will *ALWAYS* load the entity eagerly.

SQL Schema considerations
~~~~~~~~~~~~~~~~~~~~~~~~~

对于Class-Table Inheritance层次结构中的每个实体，所有映射的字段都必须是此实体的表上的列。此外，每个子表必须具有与根表上的id列定义匹配的id列（任何序列或自动增量详细信息除外）。此外，每个子表都必须有一个从id列指向根表id列的外键，并在删除时级联。
For each entity in the Class-Table Inheritance hierarchy all the
mapped fields have to be columns on the table of this entity.
Additionally each child table has to have an id column that matches
the id column definition on the root table (except for any sequence
or auto-increment details). Furthermore each child table has to
have a foreign key pointing from the id column to the root table id
column and cascading on delete.

.. _inheritence_mapping_overrides:

重写
---------

用于覆盖实体字段或关系的映射。可以应用于扩展映射超类的实体，以覆盖由映射的超类定义的关系或字段映射。
Used to override a mapping for an entity field or relationship.
May be applied to an entity that extends a mapped superclass
to override a relationship or field mapping defined by the mapped superclass.


关联重写
~~~~~~~~~~~~~~~~~~~~

覆盖实体关系的映射。
Override a mapping for an entity relationship.

可以由扩展映射的超类的实体使用，以覆盖由映射的超类定义的关系映射。
Could be used by an entity that extends a mapped superclass
to override a relationship mapping defined by the mapped superclass.

示例：

.. configuration-block::

    .. code-block:: php

        <?php
        // user mapping
        namespace MyProject\Model;
        /**
         * @MappedSuperclass
         */
        class User
        {
            //other fields mapping

            /**
             * @ManyToMany(targetEntity="Group", inversedBy="users")
             * @JoinTable(name="users_groups",
             *  joinColumns={@JoinColumn(name="user_id", referencedColumnName="id")},
             *  inverseJoinColumns={@JoinColumn(name="group_id", referencedColumnName="id")}
             * )
             */
            protected $groups;

            /**
             * @ManyToOne(targetEntity="Address")
             * @JoinColumn(name="address_id", referencedColumnName="id")
             */
            protected $address;
        }

        // admin mapping
        namespace MyProject\Model;
        /**
         * @Entity
         * @AssociationOverrides({
         *      @AssociationOverride(name="groups",
         *          joinTable=@JoinTable(
         *              name="users_admingroups",
         *              joinColumns=@JoinColumn(name="adminuser_id"),
         *              inverseJoinColumns=@JoinColumn(name="admingroup_id")
         *          )
         *      ),
         *      @AssociationOverride(name="address",
         *          joinColumns=@JoinColumn(
         *              name="adminaddress_id", referencedColumnName="id"
         *          )
         *      )
         * })
         */
        class Admin extends User
        {
        }

    .. code-block:: xml

        <!-- user mapping -->
        <doctrine-mapping>
          <mapped-superclass name="MyProject\Model\User">
                <!-- other fields mapping -->
                <many-to-many field="groups" target-entity="Group" inversed-by="users">
                    <cascade>
                        <cascade-persist/>
                        <cascade-merge/>
                        <cascade-detach/>
                    </cascade>
                    <join-table name="users_groups">
                        <join-columns>
                            <join-column name="user_id" referenced-column-name="id" />
                        </join-columns>
                        <inverse-join-columns>
                            <join-column name="group_id" referenced-column-name="id" />
                        </inverse-join-columns>
                    </join-table>
                </many-to-many>
            </mapped-superclass>
        </doctrine-mapping>

        <!-- admin mapping -->
        <doctrine-mapping>
            <entity name="MyProject\Model\Admin">
                <association-overrides>
                    <association-override name="groups">
                        <join-table name="users_admingroups">
                            <join-columns>
                                <join-column name="adminuser_id"/>
                            </join-columns>
                            <inverse-join-columns>
                                <join-column name="admingroup_id"/>
                            </inverse-join-columns>
                        </join-table>
                    </association-override>
                    <association-override name="address">
                        <join-columns>
                            <join-column name="adminaddress_id" referenced-column-name="id"/>
                        </join-columns>
                    </association-override>
                </association-overrides>
            </entity>
        </doctrine-mapping>
    .. code-block:: yaml

        # user mapping
        MyProject\Model\User:
          type: mappedSuperclass
          # other fields mapping
          manyToOne:
            address:
              targetEntity: Address
              joinColumn:
                name: address_id
                referencedColumnName: id
              cascade: [ persist, merge ]
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
              cascade: [ persist, merge, detach ]

        # admin mapping
        MyProject\Model\Admin:
          type: entity
          associationOverride:
            address:
              joinColumn:
                adminaddress_id:
                  name: adminaddress_id
                  referencedColumnName: id
            groups:
              joinTable:
                name: users_admingroups
                joinColumns:
                  adminuser_id:
                    referencedColumnName: id
                inverseJoinColumns:
                  admingroup_id:
                    referencedColumnName: id


注意事项：

-  The "association override" specifies the overrides base on the property name.“关联覆盖”指定基于属性名称的覆盖。
-  This feature is available for all kind of associations. (OneToOne, OneToMany, ManyToOne, ManyToMany)此功能适用于所有类型的关联。（OneToOne，OneToMany，ManyToOne，ManyToMany）
-  The association type *CANNOT* be changed.关联类型不能更改。
-  The override could redefine the joinTables or joinColumns depending on the association type.覆盖可以根据关联类型重新定义joinTables或joinColumns。
-  The override could redefine inversedBy to reference more than one extended entity.覆盖可以重新定义inversedBy以引用多个扩展实体。
-  The override could redefine fetch to modify the fetch strategy of the extended entity.覆盖可以重新定义提取以修改扩展实体的提取策略。

属性重写
~~~~~~~~~~~~~~~~~~~~

覆盖字段的映射。
Override the mapping of a field.

可以由扩展映射超类的实体使用，以覆盖由映射的超类定义的字段映射。
Could be used by an entity that extends a mapped superclass to override a field mapping defined by the mapped superclass.

.. configuration-block::

    .. code-block:: php

        <?php
        // user mapping
        namespace MyProject\Model;
        /**
         * @MappedSuperclass
         */
        class User
        {
            /** @Id @GeneratedValue @Column(type="integer", name="user_id", length=150) */
            protected $id;

            /** @Column(name="user_name", nullable=true, unique=false, length=250) */
            protected $name;

            // other fields mapping
        }

        // guest mapping
        namespace MyProject\Model;
        /**
         * @Entity
         * @AttributeOverrides({
         *      @AttributeOverride(name="id",
         *          column=@Column(
         *              name     = "guest_id",
         *              type     = "integer",
         *              length   = 140
         *          )
         *      ),
         *      @AttributeOverride(name="name",
         *          column=@Column(
         *              name     = "guest_name",
         *              nullable = false,
         *              unique   = true,
         *              length   = 240
         *          )
         *      )
         * })
         */
        class Guest extends User
        {
        }

    .. code-block:: xml

        <!-- user mapping -->
        <doctrine-mapping>
          <mapped-superclass name="MyProject\Model\User">
                <id name="id" type="integer" column="user_id" length="150">
                    <generator strategy="AUTO"/>
                </id>
                <field name="name" column="user_name" type="string" length="250" nullable="true" unique="false" />
                <many-to-one field="address" target-entity="Address">
                    <cascade>
                        <cascade-persist/>
                        <cascade-merge/>
                    </cascade>
                    <join-column name="address_id" referenced-column-name="id"/>
                </many-to-one>
                <!-- other fields mapping -->
            </mapped-superclass>
        </doctrine-mapping>

        <!-- admin mapping -->
        <doctrine-mapping>
            <entity name="MyProject\Model\Guest">
                <attribute-overrides>
                    <attribute-override name="id">
                        <field column="guest_id" length="140"/>
                    </attribute-override>
                    <attribute-override name="name">
                        <field column="guest_name" type="string" length="240" nullable="false" unique="true" />
                    </attribute-override>
                </attribute-overrides>
            </entity>
        </doctrine-mapping>
    .. code-block:: yaml

        # user mapping
        MyProject\Model\User:
          type: mappedSuperclass
          id:
            id:
              type: integer
              column: user_id
              length: 150
              generator:
                strategy: AUTO
          fields:
            name:
              type: string
              column: user_name
              length: 250
              nullable: true
              unique: false
          #other fields mapping


        # guest mapping
        MyProject\Model\Guest:
          type: entity
          attributeOverride:
            id:
              column: guest_id
              type: integer
              length: 140
            name:
              column: guest_name
              type: string
              length: 240
              nullable: false
              unique: true

注意事项：

-  The "attribute override" specifies the overrides base on the property name.“属性覆盖”指定属性名称的覆盖。
-  The column type *CANNOT* be changed. If the column type is not equal you get a ``MappingException`` 列类型不能更改。如果列类型不相等，则得到aMappingException
-  The override can redefine all the columns except the type.覆盖可以重新定义除类型之外的所有列。

Query the Type
--------------

可能会发生查询特殊类型的实体的情况。由于没有直接访问鉴别器列，因此Doctrine提供了 INSTANCE OF构造。
It may happen that the entities of a special type should be queried. Because there
is no direct access to the discriminator column, Doctrine provides the
``INSTANCE OF`` construct.

以下示例显示了如何使用INSTANCE OF。有一个三级层次结构，其中一个基本实体NaturalPerson被扩展，Staff而基本实体又被扩展Technician。
The following example shows how to use ``INSTANCE OF``. There is a three level hierarchy
with a base entity ``NaturalPerson`` which is extended by ``Staff`` which in turn
is extended by ``Technician``.

通过此DQL可以实现查询员工而无需任何技术人员：
Querying for the staffs without getting any technicians can be achieved by this DQL:

.. code-block:: php

    <?php
    $query = $em->createQuery("SELECT staff FROM MyProject\Model\Staff staff WHERE staff NOT INSTANCE OF MyProject\Model\Technician");
    $staffs = $query->getResult();
