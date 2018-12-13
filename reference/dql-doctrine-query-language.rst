Doctrine查询语言
===========================

``DQL`` 既 **Doctrine Query Language **，这是一个 **对象查询语言** 的衍生物，与Hibernate查询语言（
``HQL``）或Java持久性查询语言（``JPQL``）非常相似。

从本质上讲，``DQL`` 为你的 *对象模型* 提供了强大的查询功能。
想象一下所有对象都存在于某个存储器中（如对象数据库）。
编写 ``DQL`` 查询时，请考虑查询该存储以选择你的对象的某个子集。

.. note::

    初学者的一个常见错误是将 ``DQL`` 误认为只是某种形式的
    ``SQL``，因此尝试在查询中使用表名和列名或将任意表连接在一起。
    你需要将DQL视为 *对象模型* 的查询语言，而不是你的 *关系模式*。

DQL是 *区分* 大小写的，除了名称空间，类和字段名称都区分大小写。

DQL查询的类型
--------------------

DQL作为查询语言具有映射到对应的SQL语句类型的 ``SELECT``、``UPDATE`` 和 ``DELETE`` 构造。
DQL中不允许使用 ``INSERT`` 语句，因为必须通过 ``EntityManager#persist()``
将实体及其关系引入到持久性上下文，以确保你的对象模型的一致性。

DQL的 ``SELECT`` 语句是一种非常强大的方法，用于检索无法通过关联访问的域模型部分。
此外，它们允许你在单个SQL ``select`` 语句中检索实体及其关联，与使用多个查询相比，这可以在性能上产生巨大差异。

DQL的 ``UPDATE`` 和 ``DELETE`` 语句提供了一种在你的域模型的实体上执行批量更改的方法。
当你无法将需要批量更新的所有受影响实体加载到内存中时，通常需要这样做。

``SELECT`` 查询
--------------

``DQL SELECT`` 子句
~~~~~~~~~~~~~~~~~

以下示例使用一个 ``age > 20`` 来选择所有用户：

.. code-block:: php

    $query = $em->createQuery('SELECT u FROM MyProject\Model\User u WHERE u.age > 20');
    $users = $query->getResult();

让我们检查一下该查询：

-  ``u`` 是一个引用 ``MyProject\Model\User`` 类的所谓的 *标识变量* 或 *别名*。
   通过将此别名放在 ``SELECT`` 子句中，我们指定希望此查询匹配的 ``User`` 类的所有实例都出现在查询结果中。
-  ``FROM`` 关键字后面跟着一个 *完全限定* 的类名，接着跟着一个标识变量或该类名的别名。
   此类指定我们查询的根，我们可以通过连接（稍后解释）和路径表达式进一步导航。
-  ``WHERE`` 子句中的 ``u.age`` 表达式是一个路径表达式。
   DQL中的路径表达式可以通过使用用于构造路径的 ``.`` 运算符来轻松识别。
   路径表达式 ``u.age`` 引用了 ``User`` 类上的 ``age`` 字段。

此查询的结果将是一个 ``User`` 对象列表，该列表包含了所有年龄大于20的用户。

结果的格式
~~~~~~~~~~~~~

``SELECT`` 子句中表达式的组合也会影响查询结果的性质。有三种情况：

**所有对象**

.. code-block:: sql

    SELECT u, p, n FROM Users u...

在这种情况下，由于 ``FROM`` 子句，结果将是一个 ``User`` 对象数组，而
``p`` 和 ``n`` 子句由于包含在 ``SELECT`` 子句中而融合(hydrated)。

**所有标量**

.. code-block:: sql

    SELECT u.name, u.address FROM Users u...

在这种情况下，结果将是一个数组的数组。在上面的示例中，结果数组的每个元素都是一个标量
``name`` 和 ``address`` 的值的数组。

你可以在查询中选择任何实体的标量。

**混合**

.. code-block:: sql

    ``SELECT u, p.quantity FROM Users u...``

此时，结果将再次是一个数组的数组，每个元素都是一个由
``User`` 对象和 ``p.quantity`` 标量值组成的数组。

允许多个 ``FROM`` 子句将导致结果数组元素循环遍历多个 ``FROM`` 子句中包含的类。

.. note::

    除非你还选中了选择（selection，就是 ``FROM`` 中的第一个实体）的根，否则你无法选择其他实体。

    例如，``SELECT p,n FROM Users u...`` 会出错，因为 ``u`` 不是 ``SELECT`` 的一部分。

    如果违反此约束，则Doctrine会抛出一个异常。

连接
~~~~~

``SELECT`` 查询可以包含联接。有两种类型的 ``JOIN``：“常规”连接和“获取”连接。

**常规联接**: 用于限制结果和/或计算聚合值。

**提取联接**: 用来获取相关实体并将它们包含在一个查询的融合结果中。

没有特殊的DQL关键字可以区分常规联接和提取联接。只要已联接实体的字段出现在聚合函数之外的DQL查询的
``SELECT`` 部分​​中，一个联接（无论是内联接还是外联接）就会成为 *提取连接*。否则就是 *常规联接*。

示例：

``address`` 的常规联接：

.. code-block:: php

    $query = $em->createQuery("SELECT u FROM User u JOIN u.address a WHERE a.city = 'Berlin'");
    $users = $query->getResult();

``address`` 的提取联接：

.. code-block:: php

    $query = $em->createQuery("SELECT u, a FROM User u JOIN u.address a WHERE a.city = 'Berlin'");
    $users = $query->getResult();

当Doctrine使用提取联接融合一个查询时，它会返回结果数组的根级别的 ``FROM`` 子句中的类。
在前面的示例中，返回了一个 ``User`` 实例数组，并获取每个用户的地址然后将其融合到 ``User#address`` 变量中。
如果你访问该地址，Doctrine不需要使用另一个查询来延迟加载该关联。

.. note::

    Doctrine允许你遍历你的域模型中所有对象之间的所有关联。
    尚未从数据库加载的对象将替换为延迟加载的 *代理实例*。
    未加载的集合也由延迟加载的实例替换，这些实例在 *首次* 访问时会获取所有已包含的对象。
    但是，依赖于延迟加载机制会导致对数据库执行许多小查询，这会显著影响应用的性能。
    **提取联接** 是在单个 ``SELECT`` 查询中为大多数或所有实体提供融合的解决方案。

命名和位置参数
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

DQL支持 *命名* 和 *位置* 参数，但是与许多SQL方言相反，位置参数用数字指定，例如 ``?1``、``?2`` 等等。
命名参数使用 ``:name1``、``:name2`` 等指定。

当在 ``Query#setParameter($param, $value)`` 中引用参数时，使用命名和位置参数 **而无需** 它们的前缀。

DQL的 ``SELECT`` 示例
~~~~~~~~~~~~~~~~~~~~~~~

本节包含大量DQL查询以及对正在发生的事情的一些解释。实际结果还取决于融合模式。

融合所有的用户实体：

.. code-block:: php

    $query = $em->createQuery('SELECT u FROM MyProject\Model\User u');
    $users = $query->getResult(); // User对象数组

检索所有 ``CmsUser`` 的ID：

.. code-block:: php

    $query = $em->createQuery('SELECT u.id FROM CmsUser u');
    $ids = $query->getResult(); // CmsUser的id数组

检索已撰写一篇文章的所有用户的ID：

.. code-block:: php

    $query = $em->createQuery('SELECT DISTINCT u.id FROM CmsArticle a JOIN a.user u');
    $ids = $query->getResult(); // CmsUser的id数组

检索所有文章并按文章的用户实例的名称对其进行排序：

.. code-block:: php

    $query = $em->createQuery('SELECT a FROM CmsArticle a JOIN a.user u ORDER BY u.name ASC');
    $articles = $query->getResult(); // CmsArticle对象数组

检索一个 ``CmsUser`` 的 ``Username`` 和 ``Name``：

.. code-block:: php

    $query = $em->createQuery('SELECT u.username, u.name FROM CmsUser u');
    $users = $query->getResult(); // ``CmsUser`` 的 ``Username`` 和 ``Name`` 值的数组：
    echo $users[0]['username'];

检索一个 ``ForumUser`` 及其单个已关联实体：

.. code-block:: php

    $query = $em->createQuery('SELECT u, a FROM ForumUser u JOIN u.avatar a');
    $users = $query->getResult(); // ForumUser对象及已加载关联的头像的数组
    echo get_class($users[0]->getAvatar());

检索一个 ``CmsUser`` 并提取连接他所拥有的所有电话号码：

.. code-block:: php

    $query = $em->createQuery('SELECT u, p FROM CmsUser u JOIN u.phonenumbers p');
    $users = $query->getResult(); // CmsUser对象及已加载关联的电话号码的数组
    $phonenumbers = $users[0]->getPhonenumbers();

在 *升序* 中融合一个结果：

.. code-block:: php

    $query = $em->createQuery('SELECT u FROM ForumUser u ORDER BY u.id ASC');
    $users = $query->getResult(); // ForumUser对象数组

或者按 *降序* 排列：

.. code-block:: php

    $query = $em->createQuery('SELECT u FROM ForumUser u ORDER BY u.id DESC');
    $users = $query->getResult(); // ForumUser对象数组

使用聚合函数：

.. code-block:: php

    $query = $em->createQuery('SELECT COUNT(u.id) FROM Entities\User u');
    $count = $query->getSingleScalarResult();

    $query = $em->createQuery('SELECT u, count(g.id) FROM Entities\User u JOIN u.groups g GROUP BY u.id');
    $result = $query->getResult();

使用 ``WHERE`` 子句和位置参数：

.. code-block:: php

    $query = $em->createQuery('SELECT u FROM ForumUser u WHERE u.id = ?1');
    $query->setParameter(1, 321);
    $users = $query->getResult(); // ForumUser对象数组

使用 ``WHERE`` 子句和命名参数：

.. code-block:: php

    $query = $em->createQuery('SELECT u FROM ForumUser u WHERE u.username = :name');
    $query->setParameter('name', 'Bob');
    $users = $query->getResult(); // ForumUser对象数组

使用 ``WHERE`` 子句中的嵌套条件：

.. code-block:: php

    $query = $em->createQuery('SELECT u FROM ForumUser u WHERE (u.username = :name OR u.username = :name2) AND u.id = :id');
    $query->setParameters(array(
        'name' => 'Bob',
        'name2' => 'Alice',
        'id' => 321,
    ));
    $users = $query->getResult(); // ForumUser对象数组

使用 ``COUNT DISTINCT``：

.. code-block:: php

    $query = $em->createQuery('SELECT COUNT(DISTINCT u.name) FROM CmsUser');
    $users = $query->getResult(); // ForumUser对象数组

使用 ``WHERE`` 子句中的算术表达式：

.. code-block:: php

    $query = $em->createQuery('SELECT u FROM CmsUser u WHERE ((u.id + 5000) * u.id + 3) < 10000000');
    $users = $query->getResult(); // ForumUser对象数组

使用 ``HIDDEN`` 关键字在 ``ORDER`` 子句中使用算术表达式来检索用户实体：

.. code-block:: php

    $query = $em->createQuery('SELECT u, u.posts_count + u.likes_count AS HIDDEN score FROM CmsUser u ORDER BY score');
    $users = $query->getResult(); // ForumUser对象数组

使用一个 ``LEFT JOIN`` 来融合所有用户ID和可选的关联文章ID：

.. code-block:: php

    $query = $em->createQuery('SELECT u.id, a.id as article_id FROM CmsUser u LEFT JOIN u.articles a');
    $results = $query->getResult(); // 用户ID和每个用户的所有article_id的数组

通过 ``WITH`` 指定的附加条件来限制一个 ``JOIN`` 子句：

.. code-block:: php

    $query = $em->createQuery("SELECT u FROM CmsUser u LEFT JOIN u.articles a WITH a.topic LIKE :foo");
    $query->setParameter('foo', '%foo%');
    $users = $query->getResult();

使用多个提取联接：

.. code-block:: php

    $query = $em->createQuery('SELECT u, a, p, c FROM CmsUser u JOIN u.articles a JOIN u.phonenumbers p JOIN a.comments c');
    $users = $query->getResult();

``WHERE`` 子句中的 ``BETWEEN``：

.. code-block:: php

    $query = $em->createQuery('SELECT u.name FROM CmsUser u WHERE u.id BETWEEN ?1 AND ?2');
    $query->setParameter(1, 123);
    $query->setParameter(2, 321);
    $usernames = $query->getResult();

``WHERE`` 子句中的DQL函数：

.. code-block:: php

    $query = $em->createQuery("SELECT u.name FROM CmsUser u WHERE TRIM(u.name) = 'someone'");
    $usernames = $query->getResult();

``IN()`` 表达式:

.. code-block:: php

    $query = $em->createQuery('SELECT u.name FROM CmsUser u WHERE u.id IN(46)');
    $usernames = $query->getResult();

    $query = $em->createQuery('SELECT u FROM CmsUser u WHERE u.id IN (1, 2)');
    $users = $query->getResult();

    $query = $em->createQuery('SELECT u FROM CmsUser u WHERE u.id NOT IN (1)');
    $users = $query->getResult();

``CONCAT()`` DQL函数：

.. code-block:: php

    $query = $em->createQuery("SELECT u.id FROM CmsUser u WHERE CONCAT(u.name, 's') = ?1");
    $query->setParameter(1, 'Jess');
    $ids = $query->getResult();

    $query = $em->createQuery('SELECT CONCAT(u.id, u.name) FROM CmsUser u WHERE u.id = ?1');
    $query->setParameter(1, 321);
    $idUsernames = $query->getResult();

带有相关子查询的 ``WHERE`` 子句中的 ``EXISTS``：

.. code-block:: php

    $query = $em->createQuery('SELECT u.id FROM CmsUser u WHERE EXISTS (SELECT p.phonenumber FROM CmsPhonenumber p WHERE p.user = u.id)');
    $ids = $query->getResult();

获取所有属于 ``$group`` 成员的用户：

.. code-block:: php

    $query = $em->createQuery('SELECT u.id FROM CmsUser u WHERE :groupId MEMBER OF u.groups');
    $query->setParameter('groupId', $group);
    $ids = $query->getResult();

获取拥有超过1个电话号码的所有用户：

.. code-block:: php

    $query = $em->createQuery('SELECT u FROM CmsUser u WHERE SIZE(u.phonenumbers) > 1');
    $users = $query->getResult();

获取所有没有电话号码的用户：

.. code-block:: php

    $query = $em->createQuery('SELECT u FROM CmsUser u WHERE u.phonenumbers IS EMPTY');
    $users = $query->getResult();

获取一个特定类型的所有实例，以用于继承层级：

.. versionadded:: 2.1

.. code-block:: php

    $query = $em->createQuery('SELECT u FROM Doctrine\Tests\Models\Company\CompanyPerson u WHERE u INSTANCE OF Doctrine\Tests\Models\Company\CompanyEmployee');
    $query = $em->createQuery('SELECT u FROM Doctrine\Tests\Models\Company\CompanyPerson u WHERE u INSTANCE OF ?1');
    $query = $em->createQuery('SELECT u FROM Doctrine\Tests\Models\Company\CompanyPerson u WHERE u NOT INSTANCE OF ?1');

获取在选定特定性别的给定网站上可见的所有用户：

.. versionadded:: 2.2

.. code-block:: php

    $query = $em->createQuery('SELECT u FROM User u WHERE u.gender IN (SELECT IDENTITY(agl.gender) FROM Site s JOIN s.activeGenderList agl WHERE s.id = ?1)');

.. versionadded:: 2.4

从2.4开始，``IDENTITY()`` DQL函数也适用于复合主键：

.. code-block:: php

    $query = $em->createQuery("SELECT IDENTITY(c.location, 'latitude') AS latitude, IDENTITY(c.location, 'longitude') AS longitude FROM Checkpoint c WHERE c.user = ?1");

在版本2.4之前，无法实现没有关联的实体之间的联接，你可以使用以下语法生成任意联接：

.. code-block:: php

    $query = $em->createQuery('SELECT u FROM User u JOIN Blacklist b WITH u.email = b.email');

.. note::
    ``WHERE``、``WITH`` 和 ``HAVING`` 子句之间的差异可能会令人困惑。

    - ``WHERE`` 应用于整个查询的结果
    - ``WITH`` 作为附加条件应用于一个联接。
      对于任意联接（``SELECT f, b FROM Foo f, Bar b WITH f.id = b.id``），
      ``WITH`` 是必需的，即使它是 ``1 = 1``
    - ``HAVING`` 应用于聚合（``GROUP BY``）后的查询结果

部分对象语法
^^^^^^^^^^^^^^^^^^^^^

默认情况下，当在Doctrine中运行一个DQL查询，并且只选择给定实体的字段的子集时，不会接收回对象。
相反，你只接收作为扁平矩形结果集的数组，这与你直接使用SQL并加入一些数据的情况类似。

如果要选择部分对象，可以使用 ``partial`` DQL关键字：

.. code-block:: php

    $query = $em->createQuery('SELECT partial u.{id, username} FROM CmsUser u');
    $users = $query->getResult(); // 部分加载的CmsUser对象数组

你也可以在联接时使用 ``partial`` 语法：

.. code-block:: php

    $query = $em->createQuery('SELECT partial u.{id, username}, partial a.{id, name} FROM CmsUser u JOIN u.articles a');
    $users = $query->getResult(); // 部分加载的CmsUser对象数组

``NEW`` 运算符语法
^^^^^^^^^^^^^^^^^^^^^

.. versionadded:: 2.4

使用 ``NEW`` 运算符，你可以直接从DQL查询构造数据传输对象（DTO）。

- 使用 ``SELECT NEW`` 时，你不需要一个指定映射的实体。
- 你可以指定任何PHP类，它只要求此类的构造函数与该 ``NEW`` 语句匹配。
- 这种方法涉及确切地确定你真正需要哪些列，并实例化一个包含具有这些参数的构造函数的数据传输对象。

如果要选择数据传输对象，则应创建一个类：

.. code-block:: php

    class CustomerDTO
    {
        public function __construct($name, $email, $city, $value = null)
        {
            // 将值绑定到对象属性。
        }
    }

然后使用 ``NEW`` DQL关键字：

.. code-block:: php

    $query = $em->createQuery('SELECT NEW CustomerDTO(c.name, e.email, a.city) FROM Customer c JOIN c.email e JOIN c.address a');
    $users = $query->getResult(); // CustomerDTO数组

.. code-block:: php

    $query = $em->createQuery('SELECT NEW CustomerDTO(c.name, e.email, a.city, SUM(o.value)) FROM Customer c JOIN c.email e JOIN c.address a JOIN c.orders o GROUP BY c');
    $users = $query->getResult(); // CustomerDTO数组

请注意，你只能将标量表达式传递给该构造函数。

使用 ``INDEX BY``
~~~~~~~~~~~~~~~~

``INDEX BY`` 构造不会直接转换为SQL，但会影响对象和数组的的融合。
在每个 ``FROM`` 和 ``JOIN`` 子句之后，你可以指定此类应在结果中编入索引的字段。
默认情况下，结果会以从 ``0`` 开始的数字键递增。
但是使用 ``INDEX BY``，你可以指定任何其他列作为结果的键，它实际上只对主键或唯一字段有意义：

.. code-block:: sql

    SELECT u.id, u.status, upper(u.name) nameUpper FROM User u INDEX BY u.id
    JOIN u.phonenumbers p INDEX BY p.phonenumber

返回一个以下类型的数组，由 ``user-id`` 和 ``phonenumber-id`` 索引：

.. code-block:: php

    array
      0 =>
        array
          1 =>
            object(stdClass)[299]
              public '__CLASS__' => string 'Doctrine\Tests\Models\CMS\CmsUser' (length=33)
              public 'id' => int 1
              ..
          'nameUpper' => string 'ROMANB' (length=6)
      1 =>
        array
          2 =>
            object(stdClass)[298]
              public '__CLASS__' => string 'Doctrine\Tests\Models\CMS\CmsUser' (length=33)
              public 'id' => int 2
              ...
          'nameUpper' => string 'JWAGE' (length=5)

``UPDATE`` 查询
--------------

DQL不仅允许使用字段名称来选择实体，你还可以使用DQL的 ``UPDATE`` 查询对一组实体执行批量更新。
如以下示例所示，一个 ``UPDATE`` 查询的语法按预期工作：

.. code-block:: sql

    UPDATE MyProject\Model\User u SET u.password = 'new' WHERE u.id IN (1, 2, 3)

只能在 ``WHERE`` 子句中使用子选择来引用相关实体。

.. warning::

    DQL ``UPDATE`` 语句直接移植到一个数据库 ``UPDATE`` 语句中，因此绕过任何锁定模式、事件，并且不增加版本列。
    已加载到持久性上下文中的实体将 *不会* 与已更新的数据库状态同步。
    建议调用 ``EntityManager#clear()`` 并检索任何受影响实体的新实例。


``DELETE`` 查询
----------------

也可以使用DQL指定 ``DELETE`` 查询，它们的语法与 ``UPDATE`` 语法一样简单：

.. code-block:: sql

    DELETE MyProject\Model\User u WHERE u.id = 4

相关实体的引用也适用相同的限制。

.. warning::

    DQL ``DELETE`` 语句直接移植到一个数据库 ``DELETE`` 语句中，因此如果它们未显式添加到查询的
    ``WHERE`` 子句中，则会绕过任何事件和版本列的检查。
    指定实体的额外删除是 *不会* 级联到相关实体的，即使已在元数据中指定。

函数/运算符/聚合
--------------------------------

可以将字段和标识值封装到聚合和DQL函数中。数字字段可以是使用数学运算的计算的一部分。

DQL函数
~~~~~~~~~~~~~

``SELECT`、`WHERE`` 和 ``HAVING`` 子句支持以下函数：

-  ``IDENTITY(single_association_path_expression [, fieldMapping])`` - 检索拥有方的关联的外键列
-  ``ABS(arithmetic_expression)``
-  ``CONCAT(str1, str2)``
-  ``CURRENT_DATE()`` - 返回当前日期
-  ``CURRENT_TIME()`` - 返回当前时间
-  ``CURRENT_TIMESTAMP()`` - 返回当前日期和时间的时间戳
-  ``LENGTH(str)`` - 返回给定字符串的长度
-  ``LOCATE(needle, haystack [, offset])`` - 找到在字符串中子字符串第一次出现的位置。
-  ``LOWER(str)`` - 返回小写的字符串。
-  ``MOD(a, b)`` - 返回 `a` 除以 `b` 后的余数
-  ``SIZE(collection)`` - 返回指定集合中的元素数
-  ``SQRT(q)`` - 返回 `q` 的平方根。
-  ``SUBSTRING(str, start [, length])`` - 返回给定字符串的子字符串。
-  ``TRIM([LEADING | TRAILING | BOTH] ['trchar' FROM] str)`` - 按给定的 ``trim char`` 修剪字符串，默认为空格。
-  ``UPPER(str)`` - 返回给定字符串的大写字母。
-  ``DATE_ADD(date, days, unit)`` - 添加给定日期的天数。（支持的单位是``DAY``、``MONTH``）
-  ``DATE_SUB(date, days, unit)`` - 减去给定日期的天数。（支持的单位是``DAY``、``MONTH``）
-  ``DATE_DIFF(date1, date2)`` - 计算 ``date1`` 与 ``date2`` 之间的天数差异。

算术运算符
~~~~~~~~~~~~~~~~~~~~

你可以使用数值在DQL中进行数学运算，例如：

.. code-block:: sql

    SELECT person.salary * 1.5 FROM CompanyPerson person WHERE person.salary < 100000

聚合函数
~~~~~~~~~~~~~~~~~~~

``SELECT`` 和 ``GROUP BY`` 子句中允许使用以下聚合函数：
``AVG``、``COUNT``、``MIN``、``MAX``、``SUM``。

其他表达式
~~~~~~~~~~~~~~~~~

DQL对SQL中已知的各种其他表达式提供了支持，以下是所有受支持构造的列表：

-  ``ALL/ANY/SOME`` - 在 ``WHERE`` 子句后面跟着一个子选择中使用，这类似于SQL中的等效构造。
-  ``BETWEEN a AND b`` 和 ``NOT BETWEEN a AND b`` 可用于匹配算术值的范围
-  ``IN (x1, x2, ...)`` 和 ``NOT IN (x1, x2, ..)`` 可以用来匹配一组给定的值
-  ``LIKE ..`` 和 ``NOT LIKE ..`` 使用 ``％`` 作为一个通配符匹配字符串或文本的部分
-  ``IS NULL`` 和 ``IS NOT NULL`` 用于检查 ``NULL`` 值
-  ``EXISTS`` and ``NOT EXISTS`` 与一个子选择相结合

将自己的函数添加到DQL语言中
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

默认情况下，DQL附带的函数是底层数据库的一部分。
但是，你很可能在项目开始时就选择了一个数据库平台，而且很可能永远不会更改它。
对于这种情况，你可以使用自己的专用的平台函数来轻松扩展DQL解析器。

你可以在ORM配置中注册自定义DQL函数：

.. code-block:: php

    <?php
    $config = new \Doctrine\ORM\Configuration();
    $config->addCustomStringFunction($name, $class);
    $config->addCustomNumericFunction($name, $class);
    $config->addCustomDatetimeFunction($name, $class);

    $em = EntityManager::create($dbParams, $config);

该函数必须返回字符串、数字或 ``datetime`` 值，具体取决于已注册的函数类型。
作为示例，我们将添加一个特定于MySQL的 ``FLOOR()`` 函数。所有给定的类都必须实现该基类：

.. code-block:: php

    namespace MyProject\Query\AST;

    use \Doctrine\ORM\Query\AST\Functions\FunctionNode;
    use \Doctrine\ORM\Query\Lexer;

    class MysqlFloor extends FunctionNode
    {
        public $simpleArithmeticExpression;

        public function getSql(\Doctrine\ORM\Query\SqlWalker $sqlWalker)
        {
            return 'FLOOR(' . $sqlWalker->walkSimpleArithmeticExpression(
                $this->simpleArithmeticExpression
            ) . ')';
        }

        public function parse(\Doctrine\ORM\Query\Parser $parser)
        {
            $parser->match(Lexer::T_IDENTIFIER);
            $parser->match(Lexer::T_OPEN_PARENTHESIS);

            $this->simpleArithmeticExpression = $parser->SimpleArithmeticExpression();

            $parser->match(Lexer::T_CLOSE_PARENTHESIS);
        }
    }

我们将通过调用来注册该函数，然后使用它：

.. code-block:: php

    $config = $em->getConfiguration();
    $config->registerNumericFunction('FLOOR', 'MyProject\Query\MysqlFloor');

    $dql = "SELECT FLOOR(person.salary * 1.75) FROM CompanyPerson person";

查询已继承类
--------------------------

本节演示如何查询已继承类以及期望的结果类型。

单表继承
~~~~~~~~~~~~

`单表继承 <http://martinfowler.com/eaaCatalog/singleTableInheritance.html>`_
是一种继承映射策略，其中一个层级的 *所有* 类都映射到 *单个* 数据库表。
为了区分哪一行表示层级中的哪种类型，将使用所谓的 *鉴别器列*。

首先，我们需要设置一组要使用的实体。在这个案例中，它是一个通用的 ``Person`` 和 ``Employee`` 示例：

.. code-block:: php

    namespace Entities;

    /**
     * @Entity
     * @InheritanceType("SINGLE_TABLE")
     * @DiscriminatorColumn(name="discr", type="string")
     * @DiscriminatorMap({"person" = "Person", "employee" = "Employee"})
     */
    class Person
    {
        /**
         * @Id @Column(type="integer")
         * @GeneratedValue
         */
        protected $id;

        /**
         * @Column(type="string", length=50)
         */
        protected $name;

        // ...
    }

    /**
     * @Entity
     */
    class Employee extends Person
    {
        /**
         * @Column(type="string", length=50)
         */
        private $department;

        // ...
    }

首先请注意，为这些实体创建表的生成的SQL如下所示：

.. code-block:: sql

    CREATE TABLE Person (
        id INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL,
        name VARCHAR(50) NOT NULL,
        discr VARCHAR(255) NOT NULL,
        department VARCHAR(50) NOT NULL
    )

现在，当持久化新 ``Employee`` 实例时，它将自动为我们设置鉴别器值：

.. code-block:: php


    $employee = new \Entities\Employee();
    $employee->setName('test');
    $employee->setDepartment('testing');
    $em->persist($employee);
    $em->flush();

现在让我们运行一个简单的查询来检索我们刚创建的 ``Employee``：

.. code-block:: sql

    SELECT e FROM Entities\Employee e WHERE e.name = 'test'

如果我们检查生成的SQL，你会注意到它添加了一些特殊条件以确保我们只返回 ``Employee`` 实体：

.. code-block:: sql

    SELECT p0_.id AS id0, p0_.name AS name1, p0_.department AS department2,
           p0_.discr AS discr3 FROM Person p0_
    WHERE (p0_.name = ?) AND p0_.discr IN ('employee')

类表继承
~~~~~~~~~~~~~~~~~~~~~~~

`类表继承 <http://martinfowler.com/eaaCatalog/classTableInheritance.html>`_
是一种继承映射策略，其中一个层级中的 *每个* 类都映射到 *多个* 表：它自己的表和所有父类的表。
一个子类的表通过外键约束链接到父类的表。
Doctrine2通过在层级的最顶层表中使用一个 *鉴别器列* 来实现此策略，因为这是使用类表继承来实现多态查询的最简单方法。

类表继承的示例与单表继承相同，你只需将继承类型更改从 ``SINGLE_TABLE`` 为 ``JOINED``：

.. code-block:: php

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

现在看一下为了创建数据表而生成的SQL，你会发现一些不同之处：

.. code-block:: sql

    CREATE TABLE Person (
        id INT AUTO_INCREMENT NOT NULL,
        name VARCHAR(50) NOT NULL,
        discr VARCHAR(255) NOT NULL,
        PRIMARY KEY(id)
    ) ENGINE = InnoDB;
    CREATE TABLE Employee (
        id INT NOT NULL,
        department VARCHAR(50) NOT NULL,
        PRIMARY KEY(id)
    ) ENGINE = InnoDB;
    ALTER TABLE Employee ADD FOREIGN KEY (id) REFERENCES Person(id) ON DELETE CASCADE

-  数据分割在两个表之间
-  两个表之间存在一个外键

现在，如果要插入与我们在 ``SINGLE_TABLE`` 示例中所做的相同的
``Employee``，并运行相同的查询示例，它将生成不同的SQL：自动为你联接 ``Person`` 的信息：

.. code-block:: sql

    SELECT p0_.id AS id0, p0_.name AS name1, e1_.department AS department2,
           p0_.discr AS discr3
    FROM Employee e1_ INNER JOIN Person p0_ ON e1_.id = p0_.id
    WHERE p0_.name = ?

``Query`` 类
---------------

``Doctrine\ORM\Query`` 类的一个实例表示DQL查询。
你通过调用 ``EntityManager#createQuery($dql)`` 并传递DQL查询字符串来创建一个 ``Query`` 实例。
或者，你可以创建一个空 ``Query`` 实例并在之后调用 ``Query#setDQL($dql)``。这里有些例子：

.. code-block:: php

    // $em 为 EntityManager 的实例

    // 示例1: 床底一个 DQL 字符串
    $q = $em->createQuery('select u from MyProject\Model\User u');

    // 示例2: 使用 setDQL
    $q = $em->createQuery();
    $q->setDQL('select u from MyProject\Model\User u');

查询结果的格式
~~~~~~~~~~~~~~~~~~~~

一个DQL的 ``SELECT`` 查询结果返回的格式可能会受到所谓 ``hydration mode`` 的影响。
一个融合模式指定转换SQL结果集的一种特定方式。
每个融合模式在 ``Query`` 类上都有自己的专用方法。它们是：

-  ``Query#getResult()``: 检索 *一个* **对象** 集合。
   结果是对象的一个简单 *集合* （纯粹）或对象被嵌套在结果行中的 *数组* （混合）。
-  ``Query#getSingleResult()``: 检索 *单个* **对象**。
   如果结果包含多个对象，则抛出一个 ``NonUniqueResultException``。
   如果结果不包含任何对象，则抛出一个 ``NoResultException``。纯粹/混合的区别不适用于此。
-  ``Query#getOneOrNullResult()``: 检索 *单个* **对象**。如果没有找到对象，则返回 ``null``。
-  ``Query#getArrayResult()``: 检索 *一个* **数组** 图表（嵌套数组），该数组图表(graph)与
   ``Query#getResult()`` 为只读目的而生成的对象图表在很大程度上可以互换。

    .. note::

        由于数组和对象之间的标识语义不同，在某些情况下，一个数组图图表 *可能* 与对应的对象图表不同。

-  ``Query#getScalarResult()``: 检索可包含重复数据的 *标量值* 的一个扁平/矩形结果集。
   纯粹/混合的区别不适用于此。
-  ``Query#getSingleScalarResult()``: 从DBMS返回的结果中检索 *单个* **标量值**。
   如果该结果包含多个标量值，则抛出一个异常。纯粹/混合的区别不适用于此。

你也可以不使用这些方法，而是使用
``Query#execute(array $params = array(), $hydrationMode = Query::HYDRATE_OBJECT)``
通用方法。
使用此方法，你可以使用 ``Query`` 常量中的一个作为第二个参数来直接决定融合模式。
实际上，前面提到的方法只是 ``execute()`` 方法的快捷方式。
例如，``Query#getResult()`` 方法在内部调用 ``execute()`` 方法，并将 ``Query::HYDRATE_OBJECT`` 作为融合模式传入。

前面提到的方法的通常是优先推荐使用的，因为它能让代码更简洁。

纯粹型和混合型的结果
~~~~~~~~~~~~~~~~~~~~~~

通过 ``Query#getResult()`` 或 ``Query#getArrayResult()`` 来检索的DQL ``SELECT``
查询，其返回的结果的性质可以有两种形式：**纯粹(pure)** 和 **混合(mixed)**。
在前面的简单示例中，你已经看到了一个 *纯粹* 的只有对象的查询结果。
默认情况下，结果类型是 **纯粹** 的，但
**只要标量值（例如聚合值或不属于一个实体的其他标量值）出现在DQL查询的 ``SELECT`` 部分​​中，结果就会变成混合**。
一个混合结果具有与纯粹结果不同的结构，以适应该标量值。

一个 *纯粹* 的结果通常如下所示：

.. code-block:: php

    $dql = "SELECT u FROM User u";

    array
        [0] => Object
        [1] => Object
        [2] => Object
        ...

另一方面，一个 *混合* 结果普遍具有以下结构：

.. code-block:: php

    $dql = "SELECT u, 'some scalar string', count(g.id) AS num FROM User u JOIN u.groups g GROUP BY u.id";

    array
        [0]
            [0] => Object
            [1] => "some scalar string"
            ['num'] => 42
            // ... 更多标量值，无论是数字索引还是名称索引
        [1]
            [0] => Object
            [1] => "some scalar string"
            ['num'] => 42
            // ... 更多标量值，无论是数字索引还是名称索引

为了更好地理解混合结果，请考虑以下DQL查询：

.. code-block:: sql

    SELECT u, UPPER(u.name) nameUpper FROM MyProject\Model\User u

此查询使用返回标量值的DQL函数 ``UPPER``，并且因为 ``SELECT``
子句中现在存在一个标量值，所以我们得到一个混合结果。

混合结果的约束如下：

-  在 ``FROM`` 子句中获取的 *对象* 始终使用 ``0`` 键定位。
-  每个没有名称的 *标量* 都按查询中给出的顺序从 ``1`` 开始编号。
-  每个 *别名* 标量都以其别名作为键给出，并保留对应名称的大小写。
-  如果从 ``FROM`` 子句中获取了多个 *对象*，则它们会每行交替。

结果如下：

.. code-block:: php

    array
        array
            [0] => User (Object)
            ['nameUpper'] => "ROMAN"
        array
            [0] => User (Object)
            ['nameUpper'] => "JONATHAN"
        ...

在PHP代码中访问它：

.. code-block:: php

    foreach ($results as $row) {
        echo "Name: " . $row[0]->getName();
        echo "Name UPPER: " . $row['nameUpper'];
    }

获取多个 ``FROM`` 实体
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

如果你获取 ``FROM`` 子句中列出的多个实体，则该融合将返回迭代不同顶级实体的行。

.. code-block:: php

    $dql = "SELECT u, g FROM User u, Group g";

    array
        [0] => Object (User)
        [1] => Object (Group)
        [2] => Object (User)
        [3] => Object (Group)

融合模式
~~~~~~~~~~~~~~~

每种融合(Hydration)模式都假设结果如何返回到用户空间。你应该了解所有细节以充分利用不同的结果格式：

不同的融合模式的常量是：

-  ``Query::HYDRATE_OBJECT``
-  ``Query::HYDRATE_ARRAY``
-  ``Query::HYDRATE_SCALAR``
-  ``Query::HYDRATE_SINGLE_SCALAR``

对象融合
^^^^^^^^^^^^^^^^

对象融合将结果集融合成对象图表：

.. code-block:: php

    $query = $em->createQuery('SELECT u FROM CmsUser u');
    $users = $query->getResult(Query::HYDRATE_OBJECT);

有时，对象融合器中的行为可能会令人困惑，这就是为什么我们列出了许多假设以供参考：

- 在 ``FROM`` 子句中提取的对象作为一个集（Set）返回，这意味着每个对象只被包含在结果数组中一次。
  即使在以多次返回对象的同一行的方式，使用````JOIN`` 或 ``GROUP BY`` 时也是如此。
  如果融合器多次看到同一个对象，那么它确保该对象只返回一次。

- 如果某个对象已经存在于任何类型的先前查询的内存中，则使用前一个对象，即使该数据库可能包含更新的数据。
  来自数据库的数据被丢弃。如果前一个对象仍然是一个未加载的代理，则会发生这种情况。

此列表可能不完整。

数组融合
^^^^^^^^^^^^^^^

你可以使用数组融合来运行相同的查询，并将结果集融合成一个表示对象图表的数组：

.. code-block:: php

    $query = $em->createQuery('SELECT u FROM CmsUser u');
    $users = $query->getResult(Query::HYDRATE_ARRAY);

你也可以使用 ``getArrayResult()`` 快捷方式：

.. code-block:: php

    $users = $query->getArrayResult();

标量融合
^^^^^^^^^^^^^^^^

如果要返回一个扁平/矩形结果集而不是对象图表，可以使用标量融合：

.. code-block:: php

    $query = $em->createQuery('SELECT u FROM CmsUser u');
    $users = $query->getResult(Query::HYDRATE_SCALAR);
    echo $users[0]['u_id'];

使用标量融合对选定的字段进行以下假设：

1. 类中的字段以结果中的DQL别名为前缀。``SELECT u.name ..`` 类型的查询会在结果行中返回一个``u_name`` 键。

单标量融合
^^^^^^^^^^^^^^^^^^^^^^^

如果你的查询只返回单个标量值，则可以使用单标量融合：

.. code-block:: php

    $query = $em->createQuery('SELECT COUNT(a.id) FROM CmsUser u LEFT JOIN u.articles a WHERE u.username = ?1 GROUP BY u.id');
    $query->setParameter(1, 'jwage');
    $numArticles = $query->getResult(Query::HYDRATE_SINGLE_SCALAR);

你也可以使用 ``getSingleScalarResult()`` 快捷方式：

.. code-block:: php

    $numArticles = $query->getSingleScalarResult();

自定义融合模式
^^^^^^^^^^^^^^^^^^^^^^

通过首先创建一个继承 ``AbstractHydrator`` 的类，你可以轻松添加你自己的自定义融合模式：

.. code-block:: php

    namespace MyProject\Hydrators;

    use Doctrine\ORM\Internal\Hydration\AbstractHydrator;

    class CustomHydrator extends AbstractHydrator
    {
        protected function _hydrateAll()
        {
            return $this->_stmt->fetchAll(PDO::FETCH_ASSOC);
        }
    }

接下来，你只需要将该类添加到ORM配置：

.. code-block:: php

    $em->getConfiguration()->addCustomHydrationMode('CustomHydrator', 'MyProject\Hydrators\CustomHydrator');

现在可以在你的查询中使用该融合器了：

.. code-block:: php

    $query = $em->createQuery('SELECT u FROM CmsUser u');
    $results = $query->getResult('CustomHydrator');

迭代大型结果集
~~~~~~~~~~~~~~~~~~~~~~~~~~~

在某些情况下，你要执行一个会返回需要处理的非常大的结果集的查询。
所有先前描述的融合模式会将结果集完全加载到内存中，这对于大型结果集可能是不可行的。
有关如何迭代大型结果集的详细信息，请参阅 `批量处理 <batch-processing.html>`_ 文档。

函数
~~~~~~~~~

下列方法对存在 ``AbstractQuery`` 类，而 ``Query`` 和 ``NativeQuery`` 都继承与它。

参数
^^^^^^^^^^

准备好使用数字或命名通配符的语句需要对数据库执行其他参数。要将参数传递给查询，可以使用以下方法：

-  ``AbstractQuery::setParameter($param, $value)`` - 将数字或命名通配符设置为给定值。
-  ``AbstractQuery::setParameters(array $params)`` - 设置一个键值对参数的数组。
-  ``AbstractQuery::getParameter($param)``
-  ``AbstractQuery::getParameters()``

命名和位置参数都传递给这些方法，但没有它们各自的的 ``?`` 或 ``:`` 前缀。

缓存相关的AP
^^^^^^^^^^^^^^^^^

你可以根据定义结果的所有变量（SQL、融合模式、参数以及提示）或用户定义的缓存键来缓存查询结果。
但是，默认情况下，查询结果根本不会缓存。你必须基于每个查询启用结果缓存。
以下示例显示了使用 ``Result Cache API`` 的完整工作流：

.. code-block:: php

    $query = $em->createQuery('SELECT u FROM MyProject\Model\User u WHERE u.id = ?1');
    $query->setParameter(1, 12);

    $query->setResultCacheDriver(new ApcCache());

    $query->useResultCache(true)
          ->setResultCacheLifeTime($seconds = 3600);

    $result = $query->getResult(); // 缓存未命中

    $query->expireResultCache(true);
    $result = $query->getResult(); // 强制到期，缓存未命中

    $query->setResultCacheId('my_query_result');
    $result = $query->getResult(); // 保存在给定的结果缓存ID中。

    // 或使用所有参数调用 useResultCache()：
    $query->useResultCache(true, $seconds = 3600, 'my_query_result');
    $result = $query->getResult(); // 缓存命中！

    // 内省
    $queryCacheProfile = $query->getQueryCacheProfile();
    $cacheDriver = $query->getResultCacheDriver();
    $lifetime = $query->getLifetime();
    $key = $query->getCacheKey();

.. note::

    你可以在 ``Doctrine\ORM\Configuration``
    实例上全局的设置结果缓存驱动器，以便将它传递给每一个 ``Query`` 和 ``NativeQuery`` 实例。

查询提示
^^^^^^^^^^^

你可以使用 ``AbstractQuery::setHint($name, $value)``
方法将提示传递给查询解析器和融合器。
目前大多数的内部查询提示都不会在用户空间(userland)中使用。
但是，以下几个提示可以在用户空间中使用：

-  ``Query::HINT_FORCE_PARTIAL_LOAD`` - 允许融合对象，尽管并非所有列都被提取。
   此查询提示可用于处理包含 ``char`` 或 ``binary`` 数据的大型结果集的内存消耗问题。
   Doctrine无法隐式重新加载这些数据。
   如果要从数据库中完全重新加载，则必须传递部分的已加载对象到 ``EntityManager::refresh()``。
-  ``Query::HINT_REFRESH`` - 此查询在 ``EntityManager::refresh()``
   内部使用，也可以在用户空间中使用。
   如果指定此提示并且一个查询返回已由 ``UnitOfWork`` 管理的实体的数据，则将刷新现有实体的字段。
   在正常操作中，会丢弃一个加载已存在实体的数据的结果集，以有利于已存在的实体。
-  ``Query::HINT_CUSTOM_TREE_WALKERS`` - 一个附加到DQL查询解析进程的额外
   ``Doctrine\ORM\Query\TreeWalker`` 实例的数组。

查询缓存（仅限DQL查询）
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

与直接执行原生SQL查询相比，解析一个DQL查询并将其转换为针对底层数据库平台的SQL查询，显然会产生一些开销。
这就是为什么有一个专用的查询缓存来缓存DQL解析器的结果。
结合使用通配符，你可以将 *生产* 中将已解析的查询数量减少到 *零*。

默认情况下，查询缓存驱动从 ``Doctrine\ORM\Configuration`` 实例传递到每个
``Doctrine\ORM\Query`` 实例，并且默认情况下也启用。
这也意味着你不需要经常使用查询缓存的参数，但是如果你要这样做，有几种方法可以与它进行交互：

-  ``Query::setQueryCacheDriver($driver)`` - 允许设置一个 ``Cache`` 实例
-  ``Query::setQueryCacheLifeTime($seconds = 3600)`` - 设置查询缓存的生命周期。
-  ``Query::expireQueryCache($bool)`` - 如果设置为 ``true``，则强制使查询缓存过期。
-  ``Query::getExpireQueryCache()``
-  ``Query::getQueryCacheDriver()``
-  ``Query::getQueryCacheLifeTime()``

首个和最大的结果单元（仅限DQL查询）
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

你可以限制从一个DQL查询返回的结果数以及指定起始偏移量，然后Doctrine使用一个操作
``select`` 查询的策略来仅返回请求的结果数：

-  ``Query::setMaxResults($maxResults)``
-  ``Query::setFirstResult($offset)``

.. note::

    如果你的查询包含一个提取联接集合，指定的结果限制方法将无法正常工作。
    设置 ``Max Results`` 来限制数据库结果行的数量，但是对于已提取联接集合，一个根实体可能出现在许多行中，能有效地融合少于指定数量的结果。

.. _dql-temporarily-change-fetch-mode:

暂时更改DQL中的提取模式
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

虽然通常你的所有关联都标记为延迟或者超级(extra)延迟，但是你会遇到使用DQL并且不希望提取联接第二、第三或第四级的实体到结果中的情况，因为SQL ``JOIN`` 的成本增加了。
你可以标记一个临时提取的多对一或一对一关联，以使用 ``WHERE .. IN`` 查询批量提取这些实体。

.. code-block:: php

    $query = $em->createQuery("SELECT u FROM MyProject\User u");
    $query->setFetchMode("MyProject\User", "address", \Doctrine\ORM\Mapping\ClassMetadata::FETCH_EAGER);
    $query->execute();

鉴于数据库中有 ``10`` 个用户和相应的地址，执行的查询将类似于：

.. code-block:: sql

    SELECT * FROM users;
    SELECT * FROM address WHERE id IN (1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

.. note::

    在查询期间更改提取模式通常对于一对一和多对一关系是有意义的。
    在这种情况下，在加载根实体（在上面的示例中的 ``user``）之后，所有必需的ID都可用。
    因此，可以执行每个关联的一个查询以获取所有已引用的实体（``address``）。

    对于一对多关系，将提取模式更改为 ``eager`` 将导致 **为每个已加载的根实体** 执行一个查询。
    这对于 ``lazy`` 提取模式没有任何改进，一旦访问它们也将逐个初始化关联。

EBNF
----

以一个 ``EBNF`` 变体编写的以下无上下文语法(grammar)描述了Doctrine查询语言。
每当你不确定DQL的可能性或特定查询的正确语法应该是什么时，你可以查阅此语法。

文档语法
~~~~~~~~~~~~~~~~

-  非终端以一个大写字符开头
-  终端以一个小写字符开头
-  括号 ``(...)`` 用于分组
-  方括号 ``[...]`` 用于定义可选部分，例如零次或一次
-  大括号 ``{...}`` 用于重复，例如零次或多次
-  双引号 ``“...”`` 用于定义一个终端字符串
-  竖条 ``|`` 代表另一种选择

终端
~~~~~~~~~

-  identifier (``name``, ``email``, ...) 必须匹配 ``[a-z_][a-z0-9_]*``
-  fully_qualified_name (``Doctrine\Tests\Models\CMS\CmsUser``) 匹配PHP的完全限定类名
-  aliased_name (``CMS:CmsUser``) 使用两个标识符，一个用于命名空间别名，另一个用于其中的类
-  string (``foo``, ``bars house``, ``%ninja%``, ...)
-  char (``/``, ``\\``, `` ``, ...)
-  integer (``-1``, ``0``, ``1``, ``34``, ...)
-  float (``-0.23``, ``0.007``, ``1.245342E+8``, ...)
-  boolean (``false``, ``true``)

查询语言
~~~~~~~~~~~~~~

.. code-block:: php

    QueryLanguage ::= SelectStatement | UpdateStatement | DeleteStatement

语句
~~~~~~~~~~

.. code-block:: php

    SelectStatement ::= SelectClause FromClause [WhereClause] [GroupByClause] [HavingClause] [OrderByClause]
    UpdateStatement ::= UpdateClause [WhereClause]
    DeleteStatement ::= DeleteClause [WhereClause]

标识符
~~~~~~~~~~~

.. code-block:: php

    /* Alias Identification usage (the "u" of "u.name") */
    IdentificationVariable ::= identifier

    /* Alias Identification declaration (the "u" of "FROM User u") */
    AliasIdentificationVariable :: = identifier

    /* identifier that must be a class name (the "User" of "FROM User u"), possibly as a fully qualified class name or namespace-aliased */
    AbstractSchemaName ::= fully_qualified_name | aliased_name | identifier

    /* Alias ResultVariable declaration (the "total" of "COUNT(*) AS total") */
    AliasResultVariable = identifier

    /* ResultVariable identifier usage of mapped field aliases (the "total" of "COUNT(*) AS total") */
    ResultVariable = identifier

    /* identifier that must be a field (the "name" of "u.name") */
    /* This is responsible to know if the field exists in Object, no matter if it's a relation or a simple field */
    FieldIdentificationVariable ::= identifier

    /* identifier that must be a collection-valued association field (to-many) (the "Phonenumbers" of "u.Phonenumbers") */
    CollectionValuedAssociationField ::= FieldIdentificationVariable

    /* identifier that must be a single-valued association field (to-one) (the "Group" of "u.Group") */
    SingleValuedAssociationField ::= FieldIdentificationVariable

    /* identifier that must be an embedded class state field */
    EmbeddedClassStateField ::= FieldIdentificationVariable

    /* identifier that must be a simple state field (name, email, ...) (the "name" of "u.name") */
    /* The difference between this and FieldIdentificationVariable is only semantical, because it points to a single field (not mapping to a relation) */
    SimpleStateField ::= FieldIdentificationVariable

路径表达式
~~~~~~~~~~~~~~~~

.. code-block:: php

    /* "u.Group" or "u.Phonenumbers" declarations */
    JoinAssociationPathExpression             ::= IdentificationVariable "." (CollectionValuedAssociationField | SingleValuedAssociationField)

    /* "u.Group" or "u.Phonenumbers" usages */
    AssociationPathExpression                 ::= CollectionValuedPathExpression | SingleValuedAssociationPathExpression

    /* "u.name" or "u.Group" */
    SingleValuedPathExpression                ::= StateFieldPathExpression | SingleValuedAssociationPathExpression

    /* "u.name" or "u.Group.name" */
    StateFieldPathExpression                  ::= IdentificationVariable "." StateField

    /* "u.Group" */
    SingleValuedAssociationPathExpression     ::= IdentificationVariable "." SingleValuedAssociationField

    /* "u.Group.Permissions" */
    CollectionValuedPathExpression            ::= IdentificationVariable "." CollectionValuedAssociationField

    /* "name" */
    StateField                                ::= {EmbeddedClassStateField "."}* SimpleStateField

子句
~~~~~~~

.. code-block:: php

    SelectClause        ::= "SELECT" ["DISTINCT"] SelectExpression {"," SelectExpression}*
    SimpleSelectClause  ::= "SELECT" ["DISTINCT"] SimpleSelectExpression
    UpdateClause        ::= "UPDATE" AbstractSchemaName ["AS"] AliasIdentificationVariable "SET" UpdateItem {"," UpdateItem}*
    DeleteClause        ::= "DELETE" ["FROM"] AbstractSchemaName ["AS"] AliasIdentificationVariable
    FromClause          ::= "FROM" IdentificationVariableDeclaration {"," IdentificationVariableDeclaration}*
    SubselectFromClause ::= "FROM" SubselectIdentificationVariableDeclaration {"," SubselectIdentificationVariableDeclaration}*
    WhereClause         ::= "WHERE" ConditionalExpression
    HavingClause        ::= "HAVING" ConditionalExpression
    GroupByClause       ::= "GROUP" "BY" GroupByItem {"," GroupByItem}*
    OrderByClause       ::= "ORDER" "BY" OrderByItem {"," OrderByItem}*
    Subselect           ::= SimpleSelectClause SubselectFromClause [WhereClause] [GroupByClause] [HavingClause] [OrderByClause]

单元
~~~~~

.. code-block:: php

    UpdateItem  ::= SingleValuedPathExpression "=" NewValue
    OrderByItem ::= (SimpleArithmeticExpression | SingleValuedPathExpression | ScalarExpression | ResultVariable | FunctionDeclaration) ["ASC" | "DESC"]
    GroupByItem ::= IdentificationVariable | ResultVariable | SingleValuedPathExpression
    NewValue    ::= SimpleArithmeticExpression | "NULL"

``From``、``Join`` 以及 ``Index by``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: php

    IdentificationVariableDeclaration          ::= RangeVariableDeclaration [IndexBy] {Join}*
    SubselectIdentificationVariableDeclaration ::= IdentificationVariableDeclaration
    RangeVariableDeclaration                   ::= AbstractSchemaName ["AS"] AliasIdentificationVariable
    JoinAssociationDeclaration                 ::= JoinAssociationPathExpression ["AS"] AliasIdentificationVariable [IndexBy]
    Join                                       ::= ["LEFT" ["OUTER"] | "INNER"] "JOIN" (JoinAssociationDeclaration | RangeVariableDeclaration) ["WITH" ConditionalExpression]
    IndexBy                                    ::= "INDEX" "BY" StateFieldPathExpression

Select表达式
~~~~~~~~~~~~~~~~~~

.. code-block:: php

    SelectExpression        ::= (IdentificationVariable | ScalarExpression | AggregateExpression | FunctionDeclaration | PartialObjectExpression | "(" Subselect ")" | CaseExpression | NewObjectExpression) [["AS"] ["HIDDEN"] AliasResultVariable]
    SimpleSelectExpression  ::= (StateFieldPathExpression | IdentificationVariable | FunctionDeclaration | AggregateExpression | "(" Subselect ")" | ScalarExpression) [["AS"] AliasResultVariable]
    PartialObjectExpression ::= "PARTIAL" IdentificationVariable "." PartialFieldSet
    PartialFieldSet         ::= "{" SimpleStateField {"," SimpleStateField}* "}"
    NewObjectExpression     ::= "NEW" AbstractSchemaName "(" NewObjectArg {"," NewObjectArg}* ")"
    NewObjectArg            ::= ScalarExpression | "(" Subselect ")"

条件表达式
~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: php

    ConditionalExpression       ::= ConditionalTerm {"OR" ConditionalTerm}*
    ConditionalTerm             ::= ConditionalFactor {"AND" ConditionalFactor}*
    ConditionalFactor           ::= ["NOT"] ConditionalPrimary
    ConditionalPrimary          ::= SimpleConditionalExpression | "(" ConditionalExpression ")"
    SimpleConditionalExpression ::= ComparisonExpression | BetweenExpression | LikeExpression |
                                    InExpression | NullComparisonExpression | ExistsExpression |
                                    EmptyCollectionComparisonExpression | CollectionMemberExpression |
                                    InstanceOfExpression

集合表达式
~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: php

    EmptyCollectionComparisonExpression ::= CollectionValuedPathExpression "IS" ["NOT"] "EMPTY"
    CollectionMemberExpression          ::= EntityExpression ["NOT"] "MEMBER" ["OF"] CollectionValuedPathExpression

文字值
~~~~~~~~~~~~~~

.. code-block:: php

    Literal     ::= string | char | integer | float | boolean
    InParameter ::= Literal | InputParameter

输入参数
~~~~~~~~~~~~~~~

.. code-block:: php

    InputParameter      ::= PositionalParameter | NamedParameter
    PositionalParameter ::= "?" integer
    NamedParameter      ::= ":" string

算术表达式
~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: php

    ArithmeticExpression       ::= SimpleArithmeticExpression | "(" Subselect ")"
    SimpleArithmeticExpression ::= ArithmeticTerm {("+" | "-") ArithmeticTerm}*
    ArithmeticTerm             ::= ArithmeticFactor {("*" | "/") ArithmeticFactor}*
    ArithmeticFactor           ::= [("+" | "-")] ArithmeticPrimary
    ArithmeticPrimary          ::= SingleValuedPathExpression | Literal | "(" SimpleArithmeticExpression ")"
                                   | FunctionsReturningNumerics | AggregateExpression | FunctionsReturningStrings
                                   | FunctionsReturningDatetime | IdentificationVariable | ResultVariable
                                   | InputParameter | CaseExpression

标量和类型表达式
~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: php

    ScalarExpression       ::= SimpleArithmeticExpression | StringPrimary | DateTimePrimary | StateFieldPathExpression | BooleanPrimary | CaseExpression | InstanceOfExpression
    StringExpression       ::= StringPrimary | ResultVariable | "(" Subselect ")"
    StringPrimary          ::= StateFieldPathExpression | string | InputParameter | FunctionsReturningStrings | AggregateExpression | CaseExpression
    BooleanExpression      ::= BooleanPrimary | "(" Subselect ")"
    BooleanPrimary         ::= StateFieldPathExpression | boolean | InputParameter
    EntityExpression       ::= SingleValuedAssociationPathExpression | SimpleEntityExpression
    SimpleEntityExpression ::= IdentificationVariable | InputParameter
    DatetimeExpression     ::= DatetimePrimary | "(" Subselect ")"
    DatetimePrimary        ::= StateFieldPathExpression | InputParameter | FunctionsReturningDatetime | AggregateExpression

.. note::

    部分 ``CASE`` 表达式尚未实现。

聚合表达式
~~~~~~~~~~~~~~~~~~~~~

.. code-block:: php

    AggregateExpression ::= ("AVG" | "MAX" | "MIN" | "SUM" | "COUNT") "(" ["DISTINCT"] SimpleArithmeticExpression ")"

Case表达式
~~~~~~~~~~~~~~~~

.. code-block:: php

    CaseExpression        ::= GeneralCaseExpression | SimpleCaseExpression | CoalesceExpression | NullifExpression
    GeneralCaseExpression ::= "CASE" WhenClause {WhenClause}* "ELSE" ScalarExpression "END"
    WhenClause            ::= "WHEN" ConditionalExpression "THEN" ScalarExpression
    SimpleCaseExpression  ::= "CASE" CaseOperand SimpleWhenClause {SimpleWhenClause}* "ELSE" ScalarExpression "END"
    CaseOperand           ::= StateFieldPathExpression | TypeDiscriminator
    SimpleWhenClause      ::= "WHEN" ScalarExpression "THEN" ScalarExpression
    CoalesceExpression    ::= "COALESCE" "(" ScalarExpression {"," ScalarExpression}* ")"
    NullifExpression      ::= "NULLIF" "(" ScalarExpression "," ScalarExpression ")"

其他表达式
~~~~~~~~~~~~~~~~~

``QUANTIFIED``/``BETWEEN``/``COMPARISON``/``LIKE``/``NULL``/``EXISTS``

.. code-block:: php

    QuantifiedExpression     ::= ("ALL" | "ANY" | "SOME") "(" Subselect ")"
    BetweenExpression        ::= ArithmeticExpression ["NOT"] "BETWEEN" ArithmeticExpression "AND" ArithmeticExpression
    ComparisonExpression     ::= ArithmeticExpression ComparisonOperator ( QuantifiedExpression | ArithmeticExpression )
    InExpression             ::= SingleValuedPathExpression ["NOT"] "IN" "(" (InParameter {"," InParameter}* | Subselect) ")"
    InstanceOfExpression     ::= IdentificationVariable ["NOT"] "INSTANCE" ["OF"] (InstanceOfParameter | "(" InstanceOfParameter {"," InstanceOfParameter}* ")")
    InstanceOfParameter      ::= AbstractSchemaName | InputParameter
    LikeExpression           ::= StringExpression ["NOT"] "LIKE" StringPrimary ["ESCAPE" char]
    NullComparisonExpression ::= (InputParameter | NullIfExpression | CoalesceExpression | AggregateExpression | FunctionDeclaration | IdentificationVariable | SingleValuedPathExpression | ResultVariable) "IS" ["NOT"] "NULL"
    ExistsExpression         ::= ["NOT"] "EXISTS" "(" Subselect ")"
    ComparisonOperator       ::= "=" | "<" | "<=" | "<>" | ">" | ">=" | "!="

函数
~~~~~~~~~

.. code-block:: php

    FunctionDeclaration ::= FunctionsReturningStrings | FunctionsReturningNumerics | FunctionsReturningDateTime

    FunctionsReturningNumerics ::=
            "LENGTH" "(" StringPrimary ")" |
            "LOCATE" "(" StringPrimary "," StringPrimary ["," SimpleArithmeticExpression]")" |
            "ABS" "(" SimpleArithmeticExpression ")" |
            "SQRT" "(" SimpleArithmeticExpression ")" |
            "MOD" "(" SimpleArithmeticExpression "," SimpleArithmeticExpression ")" |
            "SIZE" "(" CollectionValuedPathExpression ")" |
            "DATE_DIFF" "(" ArithmeticPrimary "," ArithmeticPrimary ")" |
            "BIT_AND" "(" ArithmeticPrimary "," ArithmeticPrimary ")" |
            "BIT_OR" "(" ArithmeticPrimary "," ArithmeticPrimary ")"

    FunctionsReturningDateTime ::=
            "CURRENT_DATE" |
            "CURRENT_TIME" |
            "CURRENT_TIMESTAMP" |
            "DATE_ADD" "(" ArithmeticPrimary "," ArithmeticPrimary "," StringPrimary ")" |
            "DATE_SUB" "(" ArithmeticPrimary "," ArithmeticPrimary "," StringPrimary ")"

    FunctionsReturningStrings ::=
            "CONCAT" "(" StringPrimary "," StringPrimary ")" |
            "SUBSTRING" "(" StringPrimary "," SimpleArithmeticExpression "," SimpleArithmeticExpression ")" |
            "TRIM" "(" [["LEADING" | "TRAILING" | "BOTH"] [char] "FROM"] StringPrimary ")" |
            "LOWER" "(" StringPrimary ")" |
            "UPPER" "(" StringPrimary ")" |
            "IDENTITY" "(" SingleValuedAssociationPathExpression {"," string} ")"
