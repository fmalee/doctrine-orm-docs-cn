关联的使用
=========================

实体之间的关联就像常规的面向对象的PHP代码使用对其他对象或对象集合的引用一样。

只有在调用 ``EntityManager#flush()`` 时，代码中关联的更改才会直接与数据库同步。

在Doctrine中使用关联时，你应该了解其他一些概念：

-  如果从一个集合中删除实体，是删除该关联，而不是实体本身。
   一个实体的集合始终仅表示与包含实体的关联，而不是实体本身。
-  更新双向关联时，Doctrine仅检查关联双方中的一方以查找这些更改。
   这被称为关联的 :doc:`拥有方 <unitofwork-associations>`。
-  一个引用许多实体的属性必须是 ``Doctrine\Common\Collections\Collection`` 接口的实例。

关联实体示例
----------------------------

我们将使用带有 ``Users`` 和 ``Comments`` 的简单评论系统作为实体来展示关联管理的示例。
请参阅以下示例中每个关联的PHP文档区块，以获取有关其类型的信息，以及它是拥有方还是从属方。

.. code-block:: php

    /** @Entity */
    class User
    {
        /** @Id @GeneratedValue @Column(type="string") */
        private $id;

        /**
         * 双向 - 多个用户都有多个喜欢的评论（拥有方）
         *
         * @ManyToMany(targetEntity="Comment", inversedBy="userFavorites")
         * @JoinTable(name="user_favorite_comments")
         */
        private $favorites;

        /**
         * 单向 - 多个用户已将多个评论标记为已阅读
         *
         * @ManyToMany(targetEntity="Comment")
         * @JoinTable(name="user_read_comments")
         */
        private $commentsRead;

        /**
         * 双向 - 一对多 (从属方)
         *
         * @OneToMany(targetEntity="Comment", mappedBy="author")
         */
        private $commentsAuthored;

        /**
         * 单向 - 多对一
         *
         * @ManyToOne(targetEntity="Comment")
         */
        private $firstComment;
    }

    /** @Entity */
    class Comment
    {
        /** @Id @GeneratedValue @Column(type="string") */
        private $id;

        /**
         * 双向 - 多个评论都被多个用户所青睐（从属方）
         *
         * @ManyToMany(targetEntity="User", mappedBy="favorites")
         */
        private $userFavorites;

        /**
         * 双向 - 多个评论由一个用户撰写（拥有方）
         *
         * @ManyToOne(targetEntity="User", inversedBy="commentsAuthored")
         */
         private $author;
    }

这两个实体生成以下MySQL模式（省略了外键定义）：

.. code-block:: sql

    CREATE TABLE User (
        id VARCHAR(255) NOT NULL,
        firstComment_id VARCHAR(255) DEFAULT NULL,
        PRIMARY KEY(id)
    ) ENGINE = InnoDB;

    CREATE TABLE Comment (
        id VARCHAR(255) NOT NULL,
        author_id VARCHAR(255) DEFAULT NULL,
        PRIMARY KEY(id)
    ) ENGINE = InnoDB;

    CREATE TABLE user_favorite_comments (
        user_id VARCHAR(255) NOT NULL,
        favorite_comment_id VARCHAR(255) NOT NULL,
        PRIMARY KEY(user_id, favorite_comment_id)
    ) ENGINE = InnoDB;

    CREATE TABLE user_read_comments (
        user_id VARCHAR(255) NOT NULL,
        comment_id VARCHAR(255) NOT NULL,
        PRIMARY KEY(user_id, comment_id)
    ) ENGINE = InnoDB;

建立关联
-------------------------

建立两个实体之间的一个关联是直截了当的。以下是一些 ``User`` 的单向关系的例子：

.. code-block:: php

    class User
    {
        // ...
        public function getReadComments() {
             return $this->commentsRead;
        }

        public function setFirstComment(Comment $c) {
            $this->firstComment = $c;
        }
    }

然后，交互代码将在以下代码段中展示（此处的 ``$em`` 是 ``EntityManager`` 的实例）：

.. code-block:: php

    $user = $em->find('User', $userId);

    // 单向多对多
    $comment = $em->find('Comment', $readCommentId);
    $user->getReadComments()->add($comment);

    $em->flush();

    // 单向多对一
    $myFirstComment = new Comment();
    $user->setFirstComment($myFirstComment);

    $em->persist($myFirstComment);
    $em->flush();

在双向关联的情况下，你必须更新关联双方的字段：

.. code-block:: php

    class User
    {
        // ..

        public function getAuthoredComments() {
            return $this->commentsAuthored;
        }

        public function getFavoriteComments() {
            return $this->favorites;
        }
    }

    class Comment
    {
        // ...

        public function getUserFavorites() {
            return $this->userFavorites;
        }

        public function setAuthor(User $author = null) {
            $this->author = $author;
        }
    }

    // 多对多
    $user->getFavorites()->add($favoriteComment);
    $favoriteComment->getUserFavorites()->add($user);

    $em->flush();

    // 双向的多对一/一对多
    $newComment = new Comment();
    $user->getAuthoredComments()->add($newComment);
    $newComment->setAuthor($user);

    $em->persist($newComment);
    $em->flush();

请注意双向关联的双方是如何更新的。之前的单向关联更易于处理。

删除关联
---------------------

删除两个实体之间的一个关联同样是直截了当的。有两种策略可以实现：根据键和根据元素。这里有些例子：

.. code-block:: php

    // 按元素删除
    $user->getComments()->removeElement($comment);
    $comment->setAuthor(null);

    $user->getFavorites()->removeElement($comment);
    $comment->getUserFavorites()->removeElement($user);

    // 按键删除
    $user->getComments()->remove($ithComment);
    $comment->setAuthor(null);

你还需要调用 ``$em->flush()`` 以永久持久化数据库中的这些更改。

请注意双向关联的双方是如何始终更新的。因此，单向关联更易于处理。

另请注意，如果在方法中使用类型提示，则必须指定一个 ``nullable`` 类型，即
``setAddress(?Address $address)``，否则 ``setAddress(null)`` 将无法删除关联。
解决这个问题的另一种方法是提供一个特殊的方法，比如 ``removeAddress()``。
这也可以提供了更好的封装，因为它隐藏了未具有一个 ``address`` 的内部含义。

使用集合时，请记住一个集合本质上是一个有序映射（就像PHP数组一样）。
这就是 ``remove`` 操作接受索引/键的原因。``removeElement`` 是一个使用 ``array_search``
的具有 ``O(n)`` 复杂度的单独方法，其中 ``n`` 是映射的大小。

.. note::

    由于Doctrine始终 *只* 查看双向关联更新的 *拥有方* ，因此写操作不必更新双向一对多或多对多关联的 *从属方* 集合。
    这种知识通常可以通过避免加载从属方集合来提高性能。

你还可以使用 ``Collections::clear()`` 方法来清除整个集合的内容。
你应该知道，使用此方法会在刷新操作时导致一个直接且优化的数据库删除或更新调用，而且无法识别已重新添加到集合中的实体。

假设你通过调用 ``$post->getTags()->clear();`` 清除了标签集合，然后调用
``$post->getTags()->add($tag)``。这将无法识别先前已添加的标签，因此将发出两个单独的数据库调用。

关联管理的方法
------------------------------

在实体类中封装适当的关联管理通常是个好主意。
这样可以更容易地正确使用类，并可以封装有关如何维护关联的详细信息。

以下代码显示了之前的 ``User`` 和 ``Comment`` 示例的更新，这些示例封装了大部分关联管理代码：

.. code-block:: php

    class User
    {
        //...
        public function markCommentRead(Comment $comment) {
            // 集合实现了 ArrayAccess
            $this->commentsRead[] = $comment;
        }

        public function addComment(Comment $comment) {
            if (count($this->commentsAuthored) == 0) {
                $this->setFirstComment($comment);
            }
            $this->comments[] = $comment;
            $comment->setAuthor($this);
        }

        private function setFirstComment(Comment $c) {
            $this->firstComment = $c;
        }

        public function addFavorite(Comment $comment) {
            $this->favorites->add($comment);
            $comment->addUserFavorite($this);
        }

        public function removeFavorite(Comment $comment) {
            $this->favorites->removeElement($comment);
            $comment->removeUserFavorite($this);
        }
    }

    class Comment
    {
        // ..

        public function addUserFavorite(User $user) {
            $this->userFavorites[] = $user;
        }

        public function removeUserFavorite(User $user) {
            $this->userFavorites->removeElement($user);
        }
    }

你发现，``addUserFavorite`` 和 ``removeUserFavorite`` 没有调用
``addFavorite`` 和 ``removeFavorite``，因此严格讲双向关联还没有完成。
但是，如果你天真地在 ``addUserFavorite`` 中添加 ``addFavorite``，你最终会得到一个无限循环，因此需要做更多的工作。
正如你所看到的，在普通（plain）OOP中进行适当的双向关联管理是一项非常重要的任务，并且将类中的所有细节封装起来可能有一定的挑战性。

.. note::

    如果你想确保你的集合被完美封装，你不应该直接从一个 ``getCollectionName()``
    方法中返回它们，而是调用 ``$collection->toArray()``。
    这样，实体的客户端程序就无法规避你在实体上实现的逻辑，从而进行关联管理。例如：

.. code-block:: php

    class User {
        public function getReadComments() {
            return $this->commentsRead->toArray();
        }
    }

但是，这将始终初始化集合，并根据该大小给出所有性能损失。
This will however always initialize the collection, with all the performance penalties given the size.
在大型集合的某些场景中，将读取访问完全隐藏在 ``EntityRepository`` 的方法后面甚至可能是个好主意。

关联管理没有单一、最好的方法。它在很大程度上取决于你的具体域模型的要求以及你的偏好。

同步双向集合
---------------------------------------

在多对多关联的情况下，作为开发人员，你有责任在对拥有方和从属方应用更改时同时同步它们的集合。
Doctrine只能保证融合的一致状态，而不是你的客户端代码。

使用上面的 ``User-Comment`` 实体，一个非常简单的例子可以展示你可能遇到的麻烦：

.. code-block:: php

    $user->getFavorites()->add($favoriteComment);
    // 未调用 $favoriteComment->getUserFavorites()->add($user);

    $user->getFavorites()->contains($favoriteComment); // TRUE
    $favoriteComment->getUserFavorites()->contains($user); // FALSE

在你的代码中有两种方法可以解决此问题：

1. 忽略双向集合的从属方的更新，*但是* 从不在改变其状态的请求中读取它们。
   在下一个请求中，Doctrine将再次融合一致的集合状态。
2. 始终通过管理关联的方法使双向集合保持同步。在确保变更一致之后直接读取这些集合。

.. _transitive-persistence:

传递持久性：级联操作
-------------------------------------------

Doctrine2通过级联某些操作提供了一个传递(transitive)持久性的机制。
每个关联到其他实体或实体的集合的关联都可以被配置为自动化将以下操作级联到相关实体：
``persist``、``remove``、``merge``、``detach``、``refresh`` 以及 ``all``。

``cascade: persist`` 的主要用例是避免将相关实体“暴露”到PHP应用中。
继续本章的 ``User-Comment`` 示例，这是在控制器中创建新用户和新评论的方式（未设置 ``cascade: persist``）：

.. code-block:: php

    $user = new User();
    $myFirstComment = new Comment();
    $user->addComment($myFirstComment);

    $em->persist($user);
    $em->persist($myFirstComment); // 必需的, 如果未设置 `cascade: persist` 的话
    $em->flush();

请注意，``Comment`` 实体是在控制器中进行了实例化。
为避免这种情况，``cascade: persist`` 允许你从控制器“隐藏” ``Comment`` 实体，而仅通过 ``User`` 实体访问它：

.. code-block:: php

    // User entity
    class User
    {
        private $id;
        private $comments;

        public function __construct()
        {
            $this->id = User::new();
            $this->comments = new ArrayCollection();
        }

        public function comment(string $text, DateTimeInterface $time) : void
        {
            $newComment = Comment::create($text, $time);
            $newComment->setUser($this);
            $this->comments->add($newComment);
        }

        // ...
    }

如果你接着设置该级联到 ``User#commentsAuthored`` 属性...

.. code-block:: php

    class User
    {
        //...
        /**
         * 双向 - 一对多（从属方）
         *
         * @OneToMany(targetEntity="Comment", mappedBy="author", cascade={"persist", "remove"})
         */
        private $commentsAuthored;
        //...
    }

...你现在可以如下所示创建一个用户和相关的评论：

.. code-block:: php

    $user = new User();
    $user->comment('Lorem ipsum', new DateTime());

    $em->persist($user);
    $em->flush();

.. note::

    ``cascade: persist`` 这个主意不是为了节省控制器中的任何代码行。
    如果你在控制器中实例化评论对象（即未如上所示设置用户实体），即使配置了
    ``cascade: persist``，你仍然需要调用 ``$myFirstComment->setUser($user);``。

多亏了 ``cascade: remove``，你可以轻松删除一个用户以及所有链接的评论，而无需循环遍历它们：

.. code-block:: php

    $user = $em->find('User', $deleteUserId);

    $em->remove($user);
    $em->flush();

.. note::

    级联操作在内存中执行。这意味着当即将执行级联操作时，集合和相关实体会被提取到内存中（即使它们被标记为延迟）。
    此方法允许为每个操作执行实体生命周期事件。

    但是，在级联中将对象图表拉入内存会导致相当大的性能开销，尤其是当已级联集合很大时。
    确保权衡你定义的每个级联操作的优点和缺点。

    相比依赖数据库级别的级联操作来完成删除操作，你可以使用
    :doc:`onDelete选项 <working-with-objects>` 来配置每个联接列。

即使自动级联很方便，也应小心使用。不要盲目地应用 ``cascade=all``
到所有关联，因为它会不必要地降低你的应用的性能。
对于每个被激活的级联操作，Doctrine还会将该操作应用于关联，无论是单个值还是集合值。

可达性持久：级联持久
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

还有其他语义适用于 *级联持久* 操作。在每次的 ``flush()``
操作期间，Doctrine会检测任何集合中是否有新实体，并且可能发生三种情况：

1. 一个集合中的标记为 ``cascade: persist`` 的新实体将由Doctrine直接持久化。
2. 一个集合中的未标记为 ``cascade: persist`` 的新实体将生成一个异常并回滚 ``flush()`` 操作。
3. 忽略没有新实体的集合。

此概念称为 **可达性持久**：只要将关联定义为 ``cascade: persist``，在已管理实体上发现的新实体就会自动持久化保存。

孤立移除
--------------

还有另一种仅在从集合中移除实体时才有意义的级联概念。
如果一个 ``A`` 类型的实体包含对私有(privately owned)的实体 ``B`` 的引用，那么如果 ``A`` 对 ``B``
的引用被移除了，则还应移除实体 ``B``，因为它不再被使用。

``OrphanRemoval`` 对 *一对一*、*一对多* 以及 *多对多* 关系起作用。

.. note::

    当使用 ``orphanRemoval=true`` 选项时，Doctrine会假设实体是 *私有* 的，并且 *不会* 被其他实体复用。
    如果你忽略了这个假设，即使你将该孤立实体分配给另一个实体，你的实体也会被Doctrine删除。

作为一个更好的例子，思考包含 ``Contacts``、``Addresses`` 和 ``StandingData`` 的一个 ``Addressbook`` 应用：

.. code-block:: php

    namespace Addressbook;

    use Doctrine\Common\Collections\ArrayCollection;

    /**
     * @Entity
     */
    class Contact
    {
        /** @Id @Column(type="integer") @GeneratedValue */
        private $id;

        /** @OneToOne(targetEntity="StandingData", orphanRemoval=true) */
        private $standingData;

        /** @OneToMany(targetEntity="Address", mappedBy="contact", orphanRemoval=true) */
        private $addresses;

        public function __construct()
        {
            $this->addresses = new ArrayCollection();
        }

        public function newStandingData(StandingData $sd)
        {
            $this->standingData = $sd;
        }

        public function removeAddress($pos)
        {
            unset($this->addresses[$pos]);
        }
    }

现在有两个说明删除引用时会发生什么的示例：

.. code-block:: php

    $contact = $em->find("Addressbook\Contact", $contactId);
    $contact->newStandingData(new StandingData("Firstname", "Lastname", "Street"));
    $contact->removeAddress(1);

    $em->flush();

在这个例子中，你不仅更改了 ``Contact`` 实体本身，而且还删除了 *standing data* 的引用以及一个 *address* 引用。
调用 ``flush()`` 时，不仅删除了该引用，而且还从数据库中删除旧的 *standing data* 和一个 *address* 实体。

.. _filtering-collections:

过滤集合
---------------------

集合具有一个 **过滤API**，允许从集合中分割部分数据。
如果尚未从数据库加载集合，则该过滤API可以在SQL级别上工作，以对大型集合进行优化访问。

.. code-block:: php

    <?php

    use Doctrine\Common\Collections\Criteria;

    $group          = $entityManager->find('Group', $groupId);
    $userCollection = $group->getUsers();

    $criteria = Criteria::create()
        ->where(Criteria::expr()->eq("birthday", "1982-02-17"))
        ->orderBy(array("username" => Criteria::ASC))
        ->setFirstResult(0)
        ->setMaxResults(20)
    ;

    $birthdayUsers = $userCollection->matching($criteria);

.. tip::

    你可以将集合切片的访问移动到实体的专用方法中。例如 ``Group#getTodaysBirthdayUsers()``。

``Criteria`` 具有一个可以在SQL和PHP集合级别上工作的限制匹配语言。
这意味着你可以交替使用集合匹配，独立于内存(in-memory)或sql支持(sql-backed)的集合。
This means you can use collection matching interchangeably, independent of in-memory or sql-backed collections.

.. code-block:: php

    use Doctrine\Common\Collections;

    class Criteria
    {
        /**
         * @return Criteria
         */
        static public function create();
        /**
         * @param Expression $where
         * @return Criteria
         */
        public function where(Expression $where);
        /**
         * @param Expression $where
         * @return Criteria
         */
        public function andWhere(Expression $where);
        /**
         * @param Expression $where
         * @return Criteria
         */
        public function orWhere(Expression $where);
        /**
         * @param array $orderings
         * @return Criteria
         */
        public function orderBy(array $orderings);
        /**
         * @param int $firstResult
         * @return Criteria
         */
        public function setFirstResult($firstResult);
        /**
         * @param int $maxResults
         * @return Criteria
         */
        public function setMaxResults($maxResults);
        public function getOrderings();
        public function getWhereExpression();
        public function getFirstResult();
        public function getMaxResults();
    }

你可以通过 ``ExpressionBuilder`` 构建表达式。它有以下方法：

* ``andX($arg1, $arg2, ...)``
* ``orX($arg1, $arg2, ...)``
* ``eq($field, $value)``
* ``gt($field, $value)``
* ``lt($field, $value)``
* ``lte($field, $value)``
* ``gte($field, $value)``
* ``neq($field, $value)``
* ``isNull($field)``
* ``in($field, array $values)``
* ``notIn($field, array $values)``
* ``contains($field, $value)``
* ``startsWith($field, $value)``
* ``endsWith($field, $value)``

.. note::

    ``Criteria`` 比较的兼容性存在限制。一个比较的值必须是标量值，或者不同后端之间的行为并不相同。
