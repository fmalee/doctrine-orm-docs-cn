Basic Mapping
=============

本指南介绍了实体和属性的基本映射。阅读完本指南后，你应该知道：

- 如何使用Doctrine创建可以保存到数据库的PHP对象;
- 如何配置表上的列与实体上的属性之间的映射;
- 什么是Doctrine映射类型;
- 定义主键以及Doctrine如何生成标识符;
- 如何在Doctrine中引用保留符号。

关联的映射将在下一章介绍：:doc:`关联映射 <association-mapping>`。

指南假设
-----------------

你应该已经 :doc:`安装和配置 <configuration>` 了Doctrine。

为数据库创建类
---------------------------------

你希望使用Doctrine在数据库中保存的每个PHP对象都称为“实体”。
术语“实体”描述了在许多独立请求上具有身份的对象。通常通过为实体分配唯一标识符来实现此标识。
在本教程中，以下 ``Message`` PHP类将作为示例实体：

.. code-block:: php

    class Message
    {
        private $id;
        private $text;
        private $postedAt;
    }

因为Doctrine是一个通用库，它只知道你的实体。
因为你将使用映射元数据来描述它们的存在和结构，映射元数据是告诉Doctrine你的实体应该如何存储在数据库中的配置。
文档通常会说“映射某些东西”，这意味着要编写描述你的实体的映射元数据。

Doctrine提供了几种不同的方法来指定对象-关系的映射元数据：

-  :doc:`文档注释 <annotations-reference>`
-  :doc:`XML <xml-mapping>`
-  :doc:`YAML <yaml-mapping>`
-  :doc:`PHP code <php-mapping>`

本手册通常会通过文档注释来展示映射元数据，但许多示例也展示了YAML和XML中的等效配置。

.. note::

    所有的元数据驱动是均等的。一旦从源（注释、xml或yaml）读取了类的元数据，它就存储在
    ``Doctrine\ORM\Mapping\ClassMetadata`` 类的实例中，并且这些实例存储在元数据高速缓存中。
    如果你没有使用元数据缓存（不推荐这么做！），那么XML驱动程序是最快的。

将我们的 ``Message`` 类标记为一个Doctrine实体是很简单的：

.. configuration-block::

    .. code-block:: php

        /** @Entity */
        class Message
        {
            //...
        }

    .. code-block:: xml

        <doctrine-mapping>
          <entity name="Message">
              <!-- ... -->
          </entity>
        </doctrine-mapping>

    .. code-block:: yaml

        Message:
          type: entity
          # ...

在没有附加信息的情况下，Doctrine希望将实体保存到与我们的案例中的 ``Message`` 类具有相同名称的表中。
你可以通过配置有关表的信息来更改此设置：

.. configuration-block::

    .. code-block:: php

        <?php
        /**
         * @Entity
         * @Table(name="message")
         */
        class Message
        {
            //...
        }

    .. code-block:: xml

        <doctrine-mapping>
          <entity name="Message" table="message">
              <!-- ... -->
          </entity>
        </doctrine-mapping>

    .. code-block:: yaml

        Message:
          type: entity
          table: message
          # ...

现在， ``Message`` 类将从 ``message`` 表中保存与获取信息。

属性映射
----------------

将一个PHP类标记为实体后的下一步是将其属性映射到一个表中的列。

要配置属性，请使用 ``@Column`` 文档注释。``type`` 属性指定要用于该字段的
:ref:`Doctrine映射类型 <reference-mapping-types>`。如果未指定类型，则将 ``string`` 用作默认值。

.. configuration-block::

    .. code-block:: php

        /** @Entity */
        class Message
        {
            /** @Column(type="integer") */
            private $id;
            /** @Column(length=140) */
            private $text;
            /** @Column(type="datetime", name="posted_at") */
            private $postedAt;
        }

    .. code-block:: xml

        <doctrine-mapping>
          <entity name="Message">
            <field name="id" type="integer" />
            <field name="text" length="140" />
            <field name="postedAt" column="posted_at" type="datetime" />
          </entity>
        </doctrine-mapping>

    .. code-block:: yaml

        Message:
          type: entity
          fields:
            id:
              type: integer
            text:
              length: 140
            postedAt:
              type: datetime
              column: posted_at

当我们没有通过 ``name`` 选项显式指定列名时，Doctrine假定字段名也是列名。这意味着：

* ``id`` 属性将使用 ``integer`` 类型映射到 ``id`` 列;
* ``text`` 属性将使用有默认映射类型映射到 ``text`` 列;
* ``postedAt`` 属性将映射到 ``posted_at`` 具有 ``datetime`` 类型的列。

列注释具有更多属性。这是一个完整的清单：

- ``type``: （可选，默认为 ``string``) 用于列的映射类型。
- ``name``: （可选，默认为字段名) 数据库中列的名称。
- ``length``: （可选，默认为 ``255``) 数据库列的长度。（仅在使用字符串值列时适用）。
- ``unique``: （可选，默认为 ``FALSE``) 数据库列是否为一个唯一键。
- ``nullable``: （可选，默认为 ``FALSE``) 数据库列是否可为空。
- ``precision``: （可选，默认为 ``0``) 十进制（精确数字）列的精度（仅适用于十进制列），这是值存储的最大位数。
- ``scale``: （可选，默认为 ``0``) 十进制（精确数字）列的小数（仅适用于十进制列），表示小数点右侧的位数，且不得大于 *precision*。
- ``columnDefinition``: （可选） 允许定义用于创建列的自定义DDL片段。
  **警告**：此类型通常会迷惑 ``SchemaTool``，并始终将该列检测为已更改。
- ``options``: （可选） 生成DDL语句时传递给底层数据库平台的键值对选项。

.. _reference-mapping-types:

Doctrine映射类型
----------------------

在 ``@Column`` 中使用的 ``type`` 选项接受任何现有Doctrine类型，甚至自己的自定义类型。
一个Doctrine类型定义PHP和SQL类型之间的转换，独立于你使用的数据库供应商。
Doctrine附带的所有映射类型都在受支持的数据库系统之间完全可移植。

例如， ``string`` Doctrine映射类型定义了从PHP字符串到SQL ``VARCHAR``
（或 ``VARCHAR2`` 等，具体取决于RDBMS品牌）的映射。以下是内置映射类型的快速概述：

-  ``string``: 将SQL ``VARCHAR`` 映射到PHP字符串的类型。
-  ``integer``: 将SQL ``INT`` 映射到PHP整数的类型。
-  ``smallint``: 将数据库 ``SMALLINT`` 映射到PHP整数的类型。
-  ``bigint``: 将数据库 ``BIGINT`` 映射到PHP字符串的类型。
-  ``boolean``: 将SQL ``BOOLEAN`` 或等效值（``TINYINT``）映射到PHP布尔值的类型。
-  ``decimal``: 将SQL ``DECIMAL`` 映射到PHP字符串的类型。
-  ``date``: 将SQL ``DATETIME`` 映射到PHP ``DateTime`` 对象的类型。
-  ``time``: 将SQL ``TIME`` 映射到PHP ``DateTime`` 对象的类型。
-  ``datetime``: 将SQL ``DATETIME`` / ``TIMESTAMP`` 映射到PHP ``DateTime`` 对象的类型。
-  ``datetimetz``: 将SQL ``DATETIME`` / ``TIMESTAMP`` 映射到带有时区的PHP ``DateTime`` 对象的类型。
-  ``text``: 将SQL ``CLOB`` 映射到PHP字符串的类型。
-  ``object``: 使用 ``serialize()`` 和 ``unserialize()`` 将SQL ``CLOB`` 映射到PHP对象的类型
-  ``array``: 使用 ``serialize()`` 和 ``unserialize()`` 将SQL ``CLOB`` 映射到PHP数组的类型
-  ``simple_array``: ``implode()`` 使用 ``implode()`` 和 ``explode()``
   以逗号（``,``）作为分隔符将SQL ``CLOB``
   映射到PHP数组的类型。**重要事项**：如果你确定你的值不能包含 ``,``，则仅使用此类型。
-  ``json_array``: 使用 ``json_encode()`` 和 ``json_decode()`` 将SQL ``CLOB`` 映射到PHP数组的类型
-  ``float``: 将SQL ``Float`` （双精度）映射到PHP ``double`` 的类型。**重要事项**：仅适用于使用小数点作为分隔符的语言环境。
-  ``guid``: 将数据库 ``GUID`` / ``UUID`` 映射到PHP字符串的类型。默认为 ``varchar``，但如果平台支持，则使用特定类型。
-  ``blob``: 将SQL ``BLOB`` 映射到PHP资源流的类型

一个教程文章显示了如何定义 :doc:`自己的自定义映射类型 <../cookbook/custom-mapping-types>`。

.. note::

    ``DateTime`` 和 ``Object`` 类型通过引用进行比较，而不是按值进行比较。
    如果引用更改，则Doctrine会更新此值，因此行为就像这些对象是不可变值对象一样。

.. warning::

    所有日期类型都假定你使用通过
    `date_default_timezone_set() <http://docs.php.net/manual/en/function.date-default-timezone-set.php>`_
    或 ``php.ini`` 配置的 ``date.timezone`` 设置的默认时区。使用不同的时区会导致麻烦和意外行为。

    如果你需要特定的时区处理，则必须在域中处理此问题，并从 ``UTC`` 中来回转换所有值。
    还有一个关于使用日期时间的
    :doc:`教程文章 <../cookbook/working-with-datetime>`，提供了实现多时区应用的提示。

标识符 / 主键
--------------------------

每个实体类都必须具有标识符/主键。你可以选择使用 ``@Id`` 注释来标识标识符字段。

.. configuration-block::

    .. code-block:: php

        class Message
        {
            /**
             * @Id
             * @Column(type="integer")
             * @GeneratedValue
             */
            private $id;
            //...
        }

    .. code-block:: xml

        <doctrine-mapping>
          <entity name="Message">
            <id name="id" type="integer">
                <generator strategy="AUTO" />
            </id>
            <!-- -->
          </entity>
        </doctrine-mapping>

    .. code-block:: yaml

        Message:
          type: entity
          id:
            id:
              type: integer
              generator:
                strategy: AUTO
          fields:
            # fields here

在大多数情况下，使用自动生成器策略（``@GeneratedValue``）就是你想要的。
它默认为当前数据库供应商喜欢的标识符生成机制：带有
``AUTO_INCREMENT`` 的MySQL，带有 ``SERIAL`` 的PostgreSQL，带有 ``Sequences`` 的Oracle等等。

标识符生成策略
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

前面的示例演示了如何使用默认的标识符生成策略，而无需在意具有自动检测策略的底层数据库。
还可以更明确地指定标识符生成策略，这允许你使用一些其他功能。

以下是可能的生成策略列表：

-  ``AUTO`` （默认值）：告诉Doctrine选择所使用的数据库平台首选的策略。
   首选策略是用于MySQL、SQLite、MsSQL和SQL Anywhere的
   ``IDENTITY`` 以及用于Oracle、PostgreSQL的 ``SEQUENCE``。此策略提供了完全的可移植性。
-  ``SEQUENCE``: 告诉Doctrine使用数据库序列进行ID生成。
   该策略目前不提供完全的可移植性。仅Oracle、PostgreSql以及SQL Anywhere支持序列。
-  ``IDENTITY``: 告诉Doctrine在数据库中使用在插入行时生成一个值的特殊标识列。
   此策略目前不提供完全可移植性，并受以下平台支持：
   MySQL/SQLite/SQL Anywhere（``AUTO_INCREMENT``），MSSQL（``IDENTITY``）和PostgreSQL（``SERIAL``）。
-  ``UUID``: 告诉Doctrine使用内置的通用唯一标识符生成器。该策略提供了完全的可移植性。
-  ``TABLE``: 告诉Doctrine使用单独的表进行ID生成。
   该策略提供了完全的可移植性。 **此策略尚未实现！**
-  ``NONE``: 告诉Doctrine由你的代码分配（并由此生成）标识符。
   分配必须在传递 ``EntityManager#persist`` 到新实体之前进行。
   ``NONE`` 与完全取消 ``@GeneratedValue`` 注释的效果相同。
-  ``CUSTOM``: 使用此选项，你可以使用 ``@CustomIdGenerator``
   注释。它会让你传递一个 :doc:`你自己的类来生成标识符 <_annref_customidgenerator>`。

序列生成器
^^^^^^^^^^^^^^^^^^

当前序列生成器可以与Oracle或Postgres结合使用，除了指定序列名称外，还允许一些其他配置选项：

.. configuration-block::

    .. code-block:: php

        <?php
        class Message
        {
            /**
             * @Id
             * @GeneratedValue(strategy="SEQUENCE")
             * @SequenceGenerator(sequenceName="message_seq", initialValue=1, allocationSize=100)
             */
            protected $id = null;
            //...
        }

    .. code-block:: xml

        <doctrine-mapping>
          <entity name="Message">
            <id name="id" type="integer">
                <generator strategy="SEQUENCE" />
                <sequence-generator sequence-name="message_seq" allocation-size="100" initial-value="1" />
            </id>
          </entity>
        </doctrine-mapping>

    .. code-block:: yaml

        Message:
          type: entity
          id:
            id:
              type: integer
              generator:
                strategy: SEQUENCE
              sequenceGenerator:
                sequenceName: message_seq
                allocationSize: 100
                initialValue: 1

``initialValue`` 指定序列应从哪个值开始。

``allocationSize`` 是优化Doctrine的 ``INSERT`` 性能的强大功能。
``allocationSize`` 指定每当检索下一个值时序列递增的值。如果该值大于
``1``，则Doctrine可以为 ``allocationSize`` 个实体生成标识符值。
在上面的例子中，``allocationSize=100`` 可以让Doctrine2只需要访问序列一次就可以生成
``100`` 个新实体的标识符。

*``@SequenceGenerator`` 的默认 ``allocationSize`` 值当前为``10``*。

.. caution::

    ``allocateSize`` 由 ``SchemaTool`` 检测并转换为 ``CREATE SEQUENCE``
    语句中的 ``INCREMENT BY`` 子句。
    对于手动创建（不是SchemaTool）的数据库模式，你必须确保 ``allocationSize``
    配置选项永远不会大于实际序列的 ``INCREMENT BY`` 值，否则你可能会得到重复的键。

.. note::

    可以使用 ``strategy =“AUTO”`` 并同时指定一个 ``@SequenceGenerator``。
    在这种情况下，你的自定义序列设置用于底层平台的首选策略是 ``是SEQUENCE``，例如Oracle和PostgreSQL。

复合键
~~~~~~~~~~~~~~

使用Doctrine2，你可以使用复合主键，即在多个列上使用 ``@Id``。
在这种情况下，存在一些与使用单个标识符相反的限制：不支持使用 ``@GeneratedValue``
注释，这意味着如果在实体调用 ``EntityManager#persist()`` 之前你自己生成了主键值，则只能使用复合键。

有关复合主键的更多详细信息，请参阅 :doc:`专用教程 <../tutorials/composite-primary-keys>`。

引用保留字
----------------------

有时，由于保留字冲突，有必要引用(quote)列或表名。
Doctrine不会自动引用标识符，因为它会导致比它解决的问题更多的问题。引用表和列的名称需要使用定义中的记号(tick)显式地完成。

.. code-block:: php

    /** @Column(name="`number`", type="integer") */
    private $number;

然后，Doctrine将根据使用的数据库平台在所有SQL语句中引用此列名。

.. warning::

    除非你使用自定义 ``QuoteStrategy``，否则标识符引用不适用于连接（join）列名称或鉴别器(discriminator)列名称。

.. _reference-basic-mapping-custom-mapping-types:

.. versionadded: 2.3

为了更好地控制列引用，在2.3中引入了 ``Doctrine\ORM\Mapping\QuoteStrategy`` 接口。
它为每个列、表、别名和其他SQL名称调用。你可以实现 ``QuoteStrategy`` 并通过调用
``Doctrine\ORM\Configuration#setQuoteStrategy()`` 来设置它。

.. versionadded: 2.4

添加了 ``ANSI`` 引用策略，该策略假定任何SQL名称都不需要引用。你可以使用以下代码：

.. code-block:: php

    use Doctrine\ORM\Mapping\AnsiQuoteStrategy;

    $configuration->setQuoteStrategy(new AnsiQuoteStrategy());
