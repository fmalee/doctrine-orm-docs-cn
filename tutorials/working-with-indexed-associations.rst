使用索引关联
=================================

.. note::

    此功能从Doctrile2.1版以后可用。

Doctrine2的集合以PHP原生数组为模型。
PHP数组是一个有序的散列映射(hashmap)，但是在Doctrine的第一个版本中，从数据库中检索的键总是数字的，除非使用 ``INDEX BY``。
从Doctrine2.1开始，你可以通过相关实体中的一个值来索引集合。这是通过Doctrine ORM实现完全有序的散列映射支持的第一步。
该功能的作用类似于所选关联的一个隐含 ``INDEX BY``，但也有几个缺点：

-  如果要按字段值更改索引，则必须同时管理键和字段。
-  在每个请求上，键都是从字段值重新生成的，而不是从前一个集合的键重新生成的。
-  在持久性过程中永远不会考虑索引(Index-By)键的值。它们仅用于访问目的。
-  用作索引功能的字段 **必须** 在数据库中是唯一的。具有相同索引字段值的多个实体的行为是未定义的。

作为示例，我们将设计一个简单的股票交易所视图。
该域包括 ``Stock`` 和 ``Market`` 实体，每个 ``Stock`` 都有一个股票代码，并且在单一市场进行交易。
不是持有在一个市场上交易的股票的数量清单，而是使用它们的股票代码进行索引，该代码在所有市场中都是独一无二的。

映射索引关联
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

你可以通过添加以下内容来映射索引关联：

    * 任何 ``@OneToMany`` 或 ``@ManyToMany`` 注释的 ``indexBy`` 属性。
    * 任何 ``<one-to-many />`` 或 ``<many-to-many />`` xml元素的 ``index-by`` 属性。
    * 在 ``manyToMany:`` 或 ``oneToMany:`` YAML映射文件定义的任何关联的 ``indexBy:`` 键-值对。

``Market`` 实体的代码和映射如下所示：

.. configuration-block::
    .. code-block:: php

        <?php
        namespace Doctrine\Tests\Models\StockExchange;

        use Doctrine\Common\Collections\ArrayCollection;

        /**
         * @Entity
         * @Table(name="exchange_markets")
         */
        class Market
        {
            /**
             * @Id @Column(type="integer") @GeneratedValue
             * @var int
             */
            private $id;

            /**
             * @Column(type="string")
             * @var string
             */
            private $name;

            /**
             * @OneToMany(targetEntity="Stock", mappedBy="market", indexBy="symbol")
             * @var Stock[]
             */
            private $stocks;

            public function __construct($name)
            {
                $this->name = $name;
                $this->stocks = new ArrayCollection();
            }

            public function getId()
            {
                return $this->id;
            }

            public function getName()
            {
                return $this->name;
            }

            public function addStock(Stock $stock)
            {
                $this->stocks[$stock->getSymbol()] = $stock;
            }

            public function getStock($symbol)
            {
                if (!isset($this->stocks[$symbol])) {
                    throw new \InvalidArgumentException("Symbol is not traded on this market.");
                }

                return $this->stocks[$symbol];
            }

            public function getStocks()
            {
                return $this->stocks->toArray();
            }
        }

    .. code-block:: xml

        <?xml version="1.0" encoding="UTF-8"?>
        <doctrine-mapping xmlns="http://doctrine-project.org/schemas/orm/doctrine-mapping"
              xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
              xsi:schemaLocation="http://doctrine-project.org/schemas/orm/doctrine-mapping
                                  http://www.doctrine-project.org/schemas/orm/doctrine-mapping.xsd">

            <entity name="Doctrine\Tests\Models\StockExchange\Market">
                <id name="id" type="integer">
                    <generator strategy="AUTO" />
                </id>

                <field name="name" type="string"/>

                <one-to-many target-entity="Stock" mapped-by="market" field="stocks" index-by="symbol" />
            </entity>
        </doctrine-mapping>

    .. code-block:: yaml

        Doctrine\Tests\Models\StockExchange\Market:
          type: entity
          id:
            id:
              type: integer
              generator:
                strategy: AUTO
          fields:
            name:
              type:string
          oneToMany:
            stocks:
              targetEntity: Stock
              mappedBy: market
              indexBy: symbol

在 ``addStock()`` 方法内部，你可以看到我们如何直接将关联的键设置为股票代码，以便我们可以在调用
``addStock()`` 后直接使用该索引关联。
在 ``getStock($symbol)`` 内部，我们通过股票代码挑选一个在特定市场上交易的股票。如果此股票不存在，则抛出一个异常。

``Stock`` 实体不包含任何新的特殊指令，但为了完整性，这里它的是代码和映射：

.. configuration-block::
    .. code-block:: php

        <?php
        namespace Doctrine\Tests\Models\StockExchange;

        /**
         * @Entity
         * @Table(name="exchange_stocks")
         */
        class Stock
        {
            /**
             * @Id @GeneratedValue @Column(type="integer")
             * @var int
             */
            private $id;

            /**
             * @Column(type="string", unique=true)
             */
            private $symbol;

            /**
             * @ManyToOne(targetEntity="Market", inversedBy="stocks")
             * @var Market
             */
            private $market;

            public function __construct($symbol, Market $market)
            {
                $this->symbol = $symbol;
                $this->market = $market;
                $market->addStock($this);
            }

            public function getSymbol()
            {
                return $this->symbol;
            }
        }

    .. code-block:: xml

        <?xml version="1.0" encoding="UTF-8"?>
        <doctrine-mapping xmlns="http://doctrine-project.org/schemas/orm/doctrine-mapping"
              xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
              xsi:schemaLocation="http://doctrine-project.org/schemas/orm/doctrine-mapping
                                  http://www.doctrine-project.org/schemas/orm/doctrine-mapping.xsd">

            <entity name="Doctrine\Tests\Models\StockExchange\Stock">
                <id name="id" type="integer">
                    <generator strategy="AUTO" />
                </id>

                <field name="symbol" type="string" unique="true" />
                <many-to-one target-entity="Market" field="market" inversed-by="stocks" />
            </entity>
        </doctrine-mapping>

    .. code-block:: yaml

        Doctrine\Tests\Models\StockExchange\Stock:
          type: entity
          id:
            id:
              type: integer
              generator:
                strategy: AUTO
          fields:
            symbol:
              type: string
          manyToOne:
            market:
              targetEntity: Market
              inversedBy: stocks

查询索引关联
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

现在我们已经定义了按股票代码来索引的股票集合，我们可以看看一些利用此索引的代码。

首先，我们将使用在单个市场上交易的两个示例股票来填充我们的数据库：

.. code-block:: php

    <?php
    // $em is the EntityManager

    $market = new Market("Some Exchange");
    $stock1 = new Stock("AAPL", $market);
    $stock2 = new Stock("GOOG", $market);

    $em->persist($market);
    $em->persist($stock1);
    $em->persist($stock2);
    $em->flush();

此代码并不特别有趣，因为尚未使用索引功能。在新请求中，我们现在可以查询该市场：

.. code-block:: php

    <?php
    // $em 为 EntityManager 的实例
    $marketId = 1;
    $symbol = "AAPL";

    $market = $em->find("Doctrine\Tests\Models\StockExchange\Market", $marketId);

    // 现在按代码来访问股票：
    $stock = $market->getStock($symbol);

    echo $stock->getSymbol(); // 将会打印 "AAPL"

``Market::addStock()`` 的实现，与 ``indexBy`` 组合，使我们能够始终通过股票代码来访问集合。
``Stock`` 是否由Doctrine管理并不重要。

这同样适用于DQL查询：``indexBy`` 配置充当连接关联的一个隐式 ``INDEX BY``。

.. code-block:: php

    <?php
    // $em 为 EntityManager 的实例
    $marketId = 1;
    $symbol = "AAPL";

    $dql = "SELECT m, s FROM Doctrine\Tests\Models\StockExchange\Market m JOIN m.stocks s WHERE m.id = ?1";
    $market = $em->createQuery($dql)
                 ->setParameter(1, $marketId)
                 ->getSingleResult();

    // 现在按代码来访问股票：
    $stock = $market->getStock($symbol);

    echo $stock->getSymbol(); // 将会打印 "AAPL"

如果要在索引关联上显式使用 ``INDEX BY``，则可以自由使用。
此外，即使关联的提取模式为 ``LAZY`` 或 ``EXTRA_LAZY``，索引关联也可以使用 ``Collection::slice()`` 函数。

展望未来
~~~~~~~~~~~~~~~~~~~~~~~

对于一个多对多关联的从属方，这里有一种方法可以将键和排序作为第三和第四参数持久化到连接表中。
在 `DDC-213 <http://www.doctrine-project.org/jira/browse/DDC-213>`_
中讨论了此功能。此功能无法在一对多关联中实现，因为它们从来不是拥有方。
