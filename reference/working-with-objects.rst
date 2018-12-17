对象的使用
====================

在本章中，我们将帮助你理解 ``EntityManager`` 和 ``UnitOfWork``。
一个工作单元类似于一个对象级的事务。初始创建一个 ``EntityManager`` 时或在调用
``EntityManager#flush()`` 之后，会隐式的启动一个新的工作单元。
可以通过调用 ``EntityManager#flush()`` 来提交一个工作单元（并启动一个新工作单元）。

可以通过调用 ``EntityManager#close()`` 来手动关闭一个工作单元。
此工作单元中尚未持久化的对象的任何更改都将丢失。

.. note::

    一定要记住：只有 ``EntityManager#flush()`` 才会永远引出对数据库执行写操作。
    任何其他方法，例如 ``EntityManager#persist($entity)`` 或
    ``EntityManager#remove($entity)``，它们仅是通知 ``UnitOfWork`` 以在刷新期间执行这些操作。

    在请求期间，不调用 ``EntityManager#flush()`` 将导致所有更改的丢失。

.. note::

    Doctrine绝不会触及实体类中的方法的公共API（如getter和setter），也不会触及构造函数方法。
    相反，它使用反射 `从/向` 你的实体对象 `获取/设置` 数据。
    当Doctrine从DB获取数据并将其保存回去时，任何放在get/set方法中的代码都不会被隐含地考虑在内。

实体和标识映射
-----------------------------

实体是具有标识的对象。他们的标识在你的域内具有概念意义。
在CMS应用中，每篇文章都有一个唯一的ID。你可以通过该ID来唯一的标识每篇文章。

以下面的示例为例，你可以在其中找到标题为 ``Hello World`` 且ID为 ``1234`` 的文章：

.. code-block:: php

    $article = $entityManager->find('CMS\Article', 1234);
    $article->setHeadline('Hello World dude!');

    $article2 = $entityManager->find('CMS\Article', 1234);
    echo $article2->getHeadline();

在这个示例中，该文章从实体管理器被访问两次，但在中间进行了修改。
Doctrine2实现了这一点，并且只会让你访问ID为 ``1234`` 的文章的一个实例，无论你多频繁的从
``EntityManager`` 中检索它，甚至无论你使用何种 ``Query`` 方法（``find``、``Repository Finder`` 或 ``DQL``）。
这称为 **标识映射** 模式，这意味着Doctrine会保留每个实体的一个映射以及每个PHP请求检索到的ID，并不断返回相同的实例。

在前面的例子中，``echo`` 会打印出 ``Hello World dude！`` 到屏幕。
你甚至可以通过运行以下代码来验证 ``$article`` 和 ``$article2`` 确实指向同一个实例：

.. code-block:: php

    if ($article === $article2) {
        echo "Yes we are the same!";
    }

有时你想要清除一个 ``EntityManager`` 的标识映射以重新开始。
我们在单元测试中定期使用这个模式来再次强制从数据库加载对象，而不是从标识映射中获取它们。
你可以调用 ``EntityManager#clear()`` 以实现此结果。

实体对象图表遍历
-----------------------------

尽管Doctrine允许你的域模型（实体类）完全分离，但在遍历关联时，永远不会出现对象“丢失”的情况。
你可以根据需要在实体模型中遍历所有关联。

以下是从新打开的 ``EntityManager`` 中获取单个 ``Article`` 实体的示例。

.. code-block:: php

    /** @Entity */
    class Article
    {
        /** @Id @Column(type="integer") @GeneratedValue */
        private $id;

        /** @Column(type="string") */
        private $headline;

        /** @ManyToOne(targetEntity="User") */
        private $author;

        /** @OneToMany(targetEntity="Comment", mappedBy="article") */
        private $comments;

        public function __construct()
        {
            $this->comments = new ArrayCollection();
        }

        public function getAuthor() { return $this->author; }
        public function getComments() { return $this->comments; }
    }

    $article = $em->find('Article', 1);

此代码仅检索 ``id`` 为 ``1`` 的 ``Article`` 实例，该实例针对数据库中的 ``articles`` 表执行单个 ``SELECT`` 语句。
你仍然可以访问已关联的 ``author`` 和 ``comments`` 属性以及它们包含的已关联对象。

这是通过利用延迟加载模式来工作的。Doctrine不是向你传回一个真正的 ``Author`` 实例和一个 ``comments``
集合，而是为你创建相应的代理实例。
只有当你第一次访问这些代理时，才会通过 ``EntityManager`` 从数据库中加载它们的状态。

这种延迟加载过程发生在幕后，隐藏在你的代码之外。请考虑以下代码：

.. code-block:: php

    $article = $em->find('Article', 1);

    // 访问用户实例的一个方法会触发延迟加载
    echo "Author: " . $article->getAuthor()->getName() . "\n";

    // 延迟加载的代理会通过 instanceof 测试：
    if ($article->getAuthor() instanceof User) {
        // 一个User代理是一个已生成的“UserProxy”类
    }

    // 作为迭代器访问 comments 会触发延迟加载
    // 使用单个SELECT语句从数据库中检索此 article 的所有 comments
    foreach ($article->getComments() as $comment) {
        echo $comment->getText() . "\n\n";
    }

    // Article::$comments 会通过 Collection 接口的 instanceof 测试
    // 但它无法通过 ArrayCollection 接口的测试
    if ($article->getComments() instanceof \Doctrine\Common\Collections\Collection) {
        echo "This will always be true!";
    }

生成的代理类代码片段看起来像下面的代码片段。一个真正的代理类会重写 *所有* 公共方法，就像如下所示的 ``getName()``：

.. code-block:: php

    class UserProxy extends User implements Proxy
    {
        private function _load()
        {
            // 延迟加载代码
        }

        public function getName()
        {
            $this->_load();
            return parent::getName();
        }
        // .. User 的其他共有方法
    }

.. warning::

    遍历延迟加载的部分的对象图表将很容易触发大量的SQL查询，并且如果习惯性很大，将会表现不佳。
    确保尽可能高效地使用DQL以提取联接所需的对象图表的所有部分。

持久化实体
-------------------

通过将一个实体传递给 ``EntityManager#persist($entity)`` 方法，可以使该实体具有持久性。
通过在某个实体上应用持久化操作，该实体变为 ``MANAGED``，这意味着它的持久性从现在开始由 ``EntityManager`` 管理。
最后，此类实体的持久状态随后将在 ``EntityManager#flush()`` 调用时与数据库正确同步。

.. note::

    在实体上调用 ``persist`` 方法不会引发在数据库上立即执行 ``SQL INSERT``。
    Doctrine应用了一种称为“事务性后写”的策略，这意味着它将延迟大多数SQL命令，直到
    ``EntityManager#flush()`` 被调用，然后发布(issue)所有必要的SQL语句，
    以便以最有效的方式将对象与数据库同步，并执行单个短事务以保持引用的完整性。

示例：

.. code-block:: php

    $user = new User;
    $user->setName('Mr.Right');
    $em->persist($user);
    $em->flush();

.. note::

    已生成实体标识符/主键在涉及相关实体的下一次成功的 ``flush`` 操作之后保证可用。
    在调用 ``persist`` 之后，你不能直接依赖已生成的标识符。反之亦然。
    在 ``flush`` 操作失败后，你不能依赖已生成的标识符。

应用于实体 ``X`` 的持久化操作的语义如下：

-  如果 ``X`` 是一个新实体，它就会被管理。
   作为 ``flush`` 操作的结果，实体 ``X`` 将被输入数据库。
-  如果 ``X`` 是一个预先存在的已管理实体，则 ``persist`` 操作会忽略它。
   但是，如果 ``X`` 与其他实体的关系是用 ``cascade=PERSIST`` 或 ``cascade=ALL``
   映射的（请参阅 :ref:`传递性持久性 <transitive-persistence>`），则
   ``persist`` 操作会级联到这些 ``X`` 引用的实体。
-  如果 ``X`` 是一个已删除实体，则它将被管理。
-  如果 ``X`` 是一个已分离实体，则会在 ``flush`` 时抛出一个异常。

删除实体
-----------------

可以通过将一个实体传递给 ``EntityManager#remove($entity)`` 方法来从持久存储中删除该实体。
通过对某个实体应用 ``remove`` 操作，该实体变为 ``REMOVED``
状态，这意味着一旦 ``EntityManager#flush()`` 被调用，其持久状态将被删除。

.. note::

    就像在实体上调用 ``persist`` 一样，``remove`` 操作不会引发在数据库上立即发布 ``SQL DELETE``。
    该实体将在下一次涉及该实体的 ``EntityManager#flush()`` 调用时被删除。
    这意味着仍可以查询计划删除的实体，并将其显示在查询和集合结果中。
    有关更多信息，请参阅 :ref:`workingobjects_database_uow_outofsync`。

实例：

.. code-block:: php

    $em->remove($user);
    $em->flush();

应用于实体 ``X`` 的 ``remove`` 操作的语义如下：

-  如果 ``X`` 是一个新实体，则 ``remove`` 操作会忽略它。
   但是，如果 ``X`` 与其他实体的关系是用 ``cascade=REMOVE`` 或 ``cascade=ALL``
   映射的（请参阅 :ref:`传递性持久性 <transitive-persistence>`），则
   ``remove`` 操作会级联到这些 ``X`` 引用的实体。
-  如果 ``X`` 是已托管实体，则 ``remove`` 操作会导致其被删除。
   如果 ``X`` 与其他实体的关系是用 ``cascade=REMOVE`` 或 ``cascade=ALL``
   映射的（请参阅 :ref:`传递性持久性 <transitive-persistence>`），则
   ``remove`` 操作会级联到这些 ``X`` 引用的实体。
-  如果 ``X`` 是已分离的实体，则抛出一个 ``InvalidArgumentException``。
-  如果 ``X`` 是已删除实体，则 ``remove`` 操作将忽略它。
-  作为 ``flush`` 操作的结果，一个已删除的实体 ``X`` 将会从数据库中删除。

删除一个实体后，其内存状态与删除前相同，已生成的标识除外。

删除实体还将自动删除链接此实体的多对多联接表中的任何现有记录。
所采取的操作取决于 ``@joinColumn`` 映射属性的 ``onDelete`` 的值。
Doctrine为每个联接表的记录发出专用 ``DELETE`` 语句，或者它取决于 ``onDelete="CASCADE"`` 的外键语义。

删除具有所有已关联对象的一个对象可以通过多种方式实现，并且具有非常不同的性能影响。

1. 如果一个关联被标记为 ``CASCADE=REMOVE``，Doctrine2将获取此关联。
   如果它是单个关联，它将把这个实体传递给 ``EntityManager#remove()``。
   如果该关联是一个集合，Doctrine将遍历其所有元素并将它们传递给 ``EntityManager#remove()``。
   在这两种情况下，级联删除语义都是递归应用的。对于大型对象图表，此删除策略可能非常昂贵。
2. 使用一个DQL ``DELETE`` 语句允许你使用单个命令删除一个类型的多个实体，而不对这些实体进行融合。
   这可以非常有效地从数据库中删除大型对象图表。
3. 使用 ``onDelete="CASCADE"`` 外键语义可以强制数据库在内部删除所有关联的对象。
   这个策略有点难以控制(right)，但它非常强大和快速。
   但是你应该知道，使用策略1（``CASCADE=REMOVE``）将完全绕过任何外键的
   ``onDelete=CASCADE`` 选项，因为Doctrine将显式地获取并删除所有关联的实体。

分离实体
------------------

要从一个 ``EntityManager`` 分离一个实体，可以调用其上的
``EntityManager#detach($entity)`` 方法，或将 ``detach`` 操作级联到它，而分离的实体将不再被它管理。
一个实体通过调用一个 ``EntityManager`` 上的 ``EntityManager#detach($entity)`` 方法分离后，因此不再通过调用其上的 ``EntityManager#detach($entity)`` 方法或通过将分离操作级联到它来进行管理。
在分离实体后，对分离的实体所做的更改（包括删除实体）将不会同步到数据库。

Doctrine不会保留对已分离实体的任何引用。

示例：

.. code-block:: php

    $em->detach($entity);

应用于实体 ``X`` 的 ``detach`` 操作的语义如下：

-  如果 ``X`` 是托管实体，则分离操作会使其分离。
   如果 ``X`` 与其他实体的关系是用 ``cascade=DETACH`` 或 ``cascade=ALL``
   映射的（请参阅 :ref:`传递性持久性 <transitive-persistence>`），则
   ``detach`` 操作会级联到这些 ``X`` 引用的实体。
-  如果 ``X`` 是新的或分离的实体，则 ``detach`` 操作将忽略它。
-  如果 ``X`` 是已删除实体，则 ``detach`` 操作将级联到 ``X`` 引用的实体，如果
   ``X`` 与其他实体的关系是用 ``cascade=DETACH`` 或 ``cascade=ALL``
   映射的（请参阅 :ref:`传递性持久性 <transitive-persistence>`），则先前引用
   ``X`` 的实体将继续引用 ``X``。

有几种情况，实体在不调用 ``detach`` 方法的情况下自动分离：

-  当 ``EntityManager#clear()`` 被调用时，当前被 ``EntityManager`` 实例管理的所有实体都会脱离。
-  当序列化一个实体时，在后续反序列化时检索的实体将被分离（对于所有被序列化并存储在某个缓存中的实体，即使用 ``Query Result Cache`` 时）。

``detach`` 操作通常不经常需要并会被 ``persist`` 和 ``remove`` 使用。

合并实体
----------------

合并实体是指将（通常是分离的）实体合并到一个 ``EntityManager`` 的上下文中，以便它们再次被管理。
要将一个实体的状态合并到 ``EntityManager`` 中，请使用 ``EntityManager#merge($entity)`` 方法。
传递的实体的状态将合并到此实体的托管副本中，随后将返回此副本。

示例：

.. code-block:: php

    $detachedEntity = unserialize($serializedEntity); // 某个已分离实体
    $entity = $em->merge($detachedEntity);
    // $entity 现在指向合并操作返回的完全托管副本。
    // $em 现在像往常一样管理 $entity 的持久性。

.. note::

    当你想要序列化/反序列化实体时，你必须使所有实体的属性变为 ``protected``，而不是 ``private``。
    原因是，如果你之前序列化了一个代理实例的类，则不会序列化私有变量并抛出一个PHP ``Notice``。

应用于实体 ``X`` 的 ``merge`` 操作的语义如下：

-  如果 ``X`` 是分离的实体，则将 ``X`` 的状态复制到具有相同标识的预先存在的管理实体实例 ``X'`` 上。
-  如果 ``X`` 是新的实体实例，则将创建一个新的托管副本 ``X'``，并将 ``X`` 的状态复制到此托管实例上。
-  如果 ``X`` 是已删除的实体实例，则将抛出一个 ``InvalidArgumentException``。
-  如果 ``X`` 是一个已托管实体，它将被合并操作忽略，但是，如果这些关系已使用级联元素值
   ``MERGE`` 或 ``ALL`` 映射，则合并操作将级联到由 ``X``
   关系引用的实体（请参阅 :ref:`传递性持久性 <transitive-persistence>`）。
-  For all entities Y referenced by relationships from X having the
   cascade element value MERGE or ALL, Y is merged recursively as Y'.
   For all such Y referenced by X, X' is set to reference Y'. (Note
   that if X is managed then X is the same object as X'.)
   对于具有级联元素值 ``MERGE`` 或 ``ALL`` 的 ``X`` 的关系引用的所有实体
   ``Y``，``Y`` 被递归地合并为 ``Y'``。对于由 ``X`` 引用的所有这样的 ``Y``，``X'``
   被设置为引用 ``Y'``。（注意，如果 ``X`` 已经被管理，则 ``X`` 与 ``X'`` 是同一个对象。）
-  If X is an entity merged to X', with a reference to another
   entity Y, where cascade=MERGE or cascade=ALL is not specified, then
   navigation of the same association from X' yields a reference to a
   managed object Y' with the same persistent identity as Y.
   如果 ``X`` 是已合并到 ``X'`` 的实体，并且引用另一个实体 ``Y``，其中未指定
   ``cascade=MERGE`` 或 ``cascade=ALL``，则从 ``X'``
   导航相同的关联将产生对托管对象 ``Y'`` 的引用。与 ``Y`` 相同的持久性身份。

``merge`` 操作将抛出一个 ``OptimisticLockException``，如果要合并的实体通过版本字段使用乐观锁定和被合并实体的版本和管理副本不匹配。这通常意味着实体在分离时已被修改。
The ``merge`` operation will throw an ``OptimisticLockException``
if the entity being merged uses optimistic locking through a
version field and the versions of the entity being merged and the
managed copy don't match. This usually means that the entity has
been modified while being detached.

``merge`` 操作通常不经常需要并且被 ``persist`` 和 ``remove`` 使用。
此操作最常见的方案是将来自某个缓存（此时已分离）的实体重新附加到一个 ``EntityManager``，并且你希望修改并持久化此类实体。

.. warning::

    If you need to perform multiple merges of entities that share certain subparts
    of their object-graphs and cascade merge, then you have to call ``EntityManager#clear()`` between the
    successive calls to ``EntityManager#merge()``. Otherwise you might end up with
    multiple copies of the "same" object in the database, however with different ids.
    如果你需要执行共享其对象图表的某些子部分和级联合并的实体的多个合并，则必须在连续调用 ``EntityManager#merge()`` 之间调用  ``EntityManager#clear()``。否则，你最终可能会在数据库中标记“相同”对象的多个副本，但具有不同的ID。

.. note::

    如果从一个缓存中加载一些已分离实体，但你不需要持久化或删除它们，或者在不需要持久性服务的情况下使用它们，则无需使用 ``merge``。也就是说，你可以直接将缓存中的已分离对象直接传递给视图。

与数据库同步
---------------------------------

持久化实体的状态与一个提交底层 ``UnitOfWork`` 的 ``EntityManager`` 刷新的数据库同步。
同步涉及将任何更新写入持久实体及其与数据库的关系。
因此，如关联映射章节中所解释的，基于关系的拥有方所持有的引用来持久保持双向关系。
The state of persistent entities is synchronized with the database
on flush of an ``EntityManager`` which commits the underlying
``UnitOfWork``. The synchronization involves writing any updates to
persistent entities and their relationships to the database.
Thereby bidirectional relationships are persisted based on the
references held by the owning side of the relationship as explained
in the Association Mapping chapter.

当 ``EntityManager#flush()`` 被调用时，Doctrine会检查所有已管理、新的和已删除实体，并执行以下操作。

.. _workingobjects_database_uow_outofsync:

数据库和工作单元不同步的影响
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

一旦你开始更改实体的状态、调用 ``persist`` 或 ``remove``，则工作单元的内容将与数据库不同步。
它们只能通过调用 ``EntityManager#flush()`` 来进行同步。
本节介绍数据库和工作单元不同步的影响。

-  仍可以从数据库中查询计划删除的实体。它们从 ``DQL`` 和 ``Repository`` 查询返回，并在集合中可见。
-  传递给 ``EntityManager#persist`` 的实体不会出现在查询结果中。
-  已更改的实体不会被数据库中的状态覆盖。这是因为标识映射将检测已存在实体的构造并假定其是最新版本。

``EntityManager#flush()`` 永远不会被Doctrine隐含地调用。你必须手动触发它。

同步新实体和已管理实体
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

``flush`` 操作适用于具有以下语义的 *已托管* 实体：

-  仅当至少一个持久字段已更改时，才使用一个SQL ``UPDATE`` 语句将实体自身与数据库同步。
-  如果实体未更改，则不会执行SQL更新。

``flush`` 操作适用于具有以下语义的 *新* 实体：

-  使用SQL ``INSERT`` 语句将实体自身与数据库同步。

对于新实体或已管理实体的所有（初始化）关系，以下语义适用于每个关联实体 ``X``：

-  如果 ``X`` 是新的并且持久化操作被配置为在关系上级联，则 ``X`` 将被持久化。
-  如果 ``X`` 是新的并且没有将持久化操作配置为在关系上级联，则会抛出一个异常以表示编程错误。
-  如果 ``X`` 已删除并且持久化操作被配置为在关系上级联，则会抛出一个异常以表示编程错误（级联将重新持久化 ``X``）。
-  如果 ``X`` 已分离并且持久化操作被配置为在关系上级联，则将抛出一个异常（这在语义上与将 ``X`` 传递给 ``persist()`` 相同）。

同步已删除的实体
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

``flush`` 操作通过从数据库中删除其持久状态来应用于已删除的实体。
在刷新时，没有级联选项与已删除实体相关，级联的 ``remove`` 选项已在执行 ``EntityManager#remove($entity)`` 期间执行。

工作单元的大小
~~~~~~~~~~~~~~~~~~~~~~~~~~

一个工作单元的大小主要是指一个特定时间点的已管理实体的数量。

刷新的消耗
~~~~~~~~~~~~~~~~~~~~

``flush`` 操作的成本主要取决于两个因素：

-  ``EntityManager`` 的当前工作单元的大小。
-  已配置的变更跟踪策略

你可以按如下方式获取工作单元的大小：

.. code-block:: php

    $uowSize = $em->getUnitOfWork()->size();

``size`` 表示工作单元中的已管理实体的数量。由于变更跟踪（请参阅“变更跟踪策略”）以及内存消耗，此大小会影响
``flush()`` 操作的性能，因此你可能希望在开发期间时不时的检查它。

.. note::

    不要在每次更改一个实体或每次调用 ``persist`` / ``remove`` / ``merge``
    等等后就调用 ``flush``，这是一种反模式，会不必要地降低应用的性能。
    相反，形成对对象进行操作的工作单元，并在完成后调用 ``flush``。
    在服务单个HTTP请求时，通常不需要调用 ``flush`` 超过0-2次。

直接访问工作单元
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

你可以通过调用 ``EntityManager#getUnitOfWork()`` 直接访问工作单元。
这将返回该 ``EntityManager`` 当前使用的 ``UnitOfWork`` 实例。

.. code-block:: php

    $uow = $em->getUnitOfWork();

.. note::

    不建议直接操作一个 ``UnitOfWork``。
    直接使用 ``UnitOfWork`` API时，请不要使用标记为 ``INTERNAL`` 的方法，并仔细阅读该API文档。

实体状态
~~~~~~~~~~~~

如架构概述中所述，实体可以处于四种可能状态之一：``NEW``、``MANAGED``、``REMOVED``、``DETACHED``。
如果你明确需要在某个 ``EntityManager`` 的上下文中找出实体的当前状态，你可以询问底层的 ``UnitOfWork``：

.. code-block:: php

    switch ($em->getUnitOfWork()->getEntityState($entity)) {
        case UnitOfWork::STATE_MANAGED:
            ...
        case UnitOfWork::STATE_REMOVED:
            ...
        case UnitOfWork::STATE_DETACHED:
            ...
        case UnitOfWork::STATE_NEW:
            ...
    }

如果实体与一个 ``EntityManager`` 相关联并且不是 ``REMOVED``，则该实体处于 ``MANAGED`` 状态。

在将实体传递给  ``EntityManager#remove()`` 之后，该实体处于 ``REMOVED``
状态，直到同一个 ``EntityManager`` 的下一次刷新操作。
在下一次刷新操作之前，一个 ``REMOVED`` 实体仍然与一个 ``EntityManager`` 相关联。

如果实体具有持久状态和标识，但当前未与一个 ``EntityManager`` 关联，则该实体处于 ``DETACHED`` 状态。

如果实体没有持久状态和标识，且当前未与一个 ``EntityManager`` 关联，（例如刚刚通过 ``new``
操作符创建的实体），则该实体处于 ``NEW`` 状态。

查询
--------

在提高功能和灵活性方面，Doctrine2提供了以下方法来查询持久对象。
你应该始终选择最适合你需求的最简单的一个。

通过主键
~~~~~~~~~~~~~~

查询一个持久对象的最基本方法是通过 *标识符/主键* 来使用
``EntityManager#find($entityName, $id)`` 方法。这是一个例子：

.. code-block:: php

    // $em 为 EntityManager 的实例
    $user = $em->find('MyProject\Domain\User', $id);

返回值是找到的实体实例，如果找不到具有给定标识符的实例，则返回 ``null``。

从本质上讲，``EntityManager#find()`` 只是以下的一个快捷方式：

.. code-block:: php

    // $em 为 EntityManager 的实例
    $user = $em->getRepository('MyProject\Domain\User')->find($id);

``EntityManager#getRepository($entityName)`` 返回一个仓库对象，它提供了许多方法来检索指定类型的实体。
默认情况下，仓库实例是 ``Doctrine\ORM\EntityRepository`` 类型。你还可以如稍后所示使用自定义仓库类。

通过简单的条件
~~~~~~~~~~~~~~~~~~~~

要根据形成逻辑连接的多个条件查询一个或多个实体，请使用一个仓库中的 ``findBy`` 和 ``findOneBy`` 方法，如下所示：

.. code-block:: php

    // $em 为 EntityManager 的实例

    // 所有20岁的用户
    $users = $em->getRepository('MyProject\Domain\User')->findBy(array('age' => 20));

    // 所有20岁且姓 'Miller' 的用户
    $users = $em->getRepository('MyProject\Domain\User')->findBy(array('age' => 20, 'surname' => 'Miller'));

    // 通过昵称检索单个用户
    $user = $em->getRepository('MyProject\Domain\User')->findOneBy(array('nickname' => 'romanb'));

你还可以通过仓库的拥有方关联来加载：

.. code-block:: php

    $number = $em->find('MyProject\Domain\Phonenumber', 1234);
    $user = $em->getRepository('MyProject\Domain\User')->findOneBy(array('phone' => $number->getId()));

``EntityRepository#findBy()`` 方法还接受排序、限制和偏移作为第二到第四参数：

.. code-block:: php

    $tenUsers = $em->getRepository('MyProject\Domain\User')->findBy(array('age' => 20), array('name' => 'ASC'), 10, 0);

如果传递一个值数组，Doctrine会自动将查询转换为一个 ``WHERE field IN (..)`` 查询：

.. code-block:: php

    $users = $em->getRepository('MyProject\Domain\User')->findBy(array('age' => array(20, 30, 40)));
    // 大致转换为：SELECT * FROM users WHERE age IN (20, 30, 40)

``EntityRepository`` 还提供了一种机制，以通过使用 ``__call`` 来实现更简洁的调用。
因此，以下两个例子是等效的：

.. code-block:: php

    // 通过昵称检索单个用户
    $user = $em->getRepository('MyProject\Domain\User')->findOneBy(array('nickname' => 'romanb'));

    // 通过昵称检索单个用户 (__call 魔术方法)
    $user = $em->getRepository('MyProject\Domain\User')->findOneByNickname('romanb');

此外，当你确实不需要数据时，你可以只统计所提供条件的结果：

.. code-block:: php

    // 检查没有昵称的用户
    $availableNickname = 0 === $em->getRepository('MyProject\Domain\User')->count(['nickname' => 'nonexistent']);

通过Criteria
~~~~~~~~~~~~~~~

.. versionadded:: 2.3

``Repository`` 实现了 ``Doctrine\Common\Collections\Selectable`` 接口。
这意味着你可以构建 ``Doctrine\Common\Collections\Criteria`` 并将它们传递给 ``matching($criteria)`` 方法。

请参阅 :doc:`关联的使用 <working-with-associations>` 文档中的 `过滤集合` 一节。

通过渴求加载
~~~~~~~~~~~~~~~~

每当你查询具有持久关联的实体并将这些关联映射为 ``EAGER`` 时，它们将自动与要查询的实体一起加载，因此可立即供你的应用使用。

通过延迟加载
~~~~~~~~~~~~~~~

每当你有一个已托管实体实例时，你可以遍历并使用已配置为 ``LAZY`` 的该实体的任何关联，就好像它们已经在内存中一样。
Doctrine将通过延迟加载的概念按需自动加载关联的对象。

通过DQL
~~~~~~~~

查询持久对象的最强大和最灵活的方法是 ``Doctrine Query Language``，它是一种对象查询语言。
DQL使你能够以对象语言来查询持久对象。
DQL了解类、字段、继承和关联。DQL在语法上与熟悉的SQL非常相似，但 *它不是SQL*。

DQL查询由 ``Doctrine\ORM\Query`` 类的实例表示。
你可以使用 ``EntityManager#createQuery($dql)`` 来创建一个查询。这是一个简单的例子：

.. code-block:: php

    // $em 为 EntityManager 的实例

    // 所有年龄在20到30岁之间的用户（包括在内）。
    $q = $em->createQuery("select u from MyDomain\Model\User u where u.age >= 20 and u.age <= 30");
    $users = $q->getResult();

请注意，此查询不包含关于关系模式的知识，仅包含有关对象模型的知识。
DQL支持位置和命名参数、许多函数、（提取）联接、聚合、子查询等等。
有关DQL及其语法以及Doctrine类的详细信息，请参阅 :doc:`专用章节 <dql-doctrine-query-language>`。
为了以编程方式构建基于仅在运行时已知的条件的查询，Doctrine提供了特殊的 ``Doctrine\ORM\QueryBuilder`` 类。
有关使用一个 ``QueryBuilder`` 来构造查询的更多信息，请参阅 :doc:`查询生成器 <query-builder>`。

通过原生查询
~~~~~~~~~~~~~~~~~

作DQL的一个替代或作为特殊SQL语句的后备，你可以使用原生查询。
原生查询是使用一个手工制作的SQL查询和一个 ``ResultSetMapping`` 来构建的，``ResultSetMapping``
描述了如何通过Doctrine转换SQL结果集。
有关原生查询的更多信息，请参阅 :doc:`专用章节 <native-sql>`。

自定义仓库
~~~~~~~~~~~~~~~~~~~

默认情况下，``EntityManager`` 会在你调用 ``EntityManager#getRepository($entityClass)``
时返回一个 ``Doctrine\ORM\EntityRepository`` 的默认实现。
你可以通过在注释、XML或YAML元数据中指定自己的实体仓库的类名来重写此行为。
在需要大量专用DQL查询的大型应用中，建议使用一个自定义仓库将这些查询分组到集中位置。

.. code-block:: php

    namespace MyDomain\Model;

    use Doctrine\ORM\EntityRepository;
    use Doctrine\ORM\Mapping as ORM;

    /**
     * @ORM\Entity(repositoryClass="MyDomain\Model\UserRepository")
     */
    class User
    {

    }

    class UserRepository extends EntityRepository
    {
        public function getAllAdminUsers()
        {
            return $this->_em->createQuery('SELECT u FROM MyDomain\Model\User u WHERE u.status = "admin"')
                             ->getResult();
        }
    }

你现在可以通过以下方式来访问你的仓库：

.. code-block:: php

    // $em 为 EntityManager 的实例

    $admins = $em->getRepository('MyDomain\Model\User')->getAllAdminUsers();
