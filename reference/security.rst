安全
========

Doctrine库的运行非常接近你的数据库，因此需要处理和预防SQL注入漏洞。

了解Doctrine如何处理安全性至关重要，因为我们无法保护你免受SQL注入。

另请阅读Doctrine DBAL中有关安全的文档章节。此页面仅处理ORM中的安全问题。

* `DBAL的安全文档 <https://github.com/doctrine/dbal/blob/master/docs/en/reference/security.rst>`_

如果你在Doctrine中发现安全漏洞，请在Jira上报告并将安全级别更改为“安全问题”。
这样的话，只有Doctrine Core开发人员才能看到它。

用户输入和Doctrine ORM
---------------------------

与单独的DBAL相比，ORM在防止SQL注入方面要好得多。你可以考虑以下API以免SQL注入：

- ``\Doctrine\ORM\EntityManager#find()`` 和 ``getReference()``.
- 通过 ``Doctrine\ORM\EntityManager#persist()`` 来插入和更新的对象上的所有的值
- 使用 ``Doctrine\ORM\EntityRepository`` 上找到的所有方法
- 通过以下方法将用户输入设置为DQL查询或 ``QueryBuilder`` 方法
    - ``setParameter()`` 或变种
    - ``setMaxResults()``
    - ``setFirstResult()``
- 在 ``Doctrine\ORM\PersistentCollection`` 和
  ``Doctrine\ORM\EntityRepository`` 上通过Criteria API进行查询

使用用户输入时，你不能安全地使用SQL注入：
You are **NOT** safe from SQL injection when using user input with:

- ``Doctrine\ORM\QueryBuilder`` 的表达式API
- 将用户输入连接到DQL的 ``SELECT``、``UPDATE``、``DELETE`` 语句或原生SQL。

这意味着在使用任何类型的查询对象时，只能使用Doctrine ORM进行SQL注入。
安全规则是：在使用一个 ``Query`` 对象时始终为用户对象使用预准备的语句参数。

.. warning::

    不要复制粘贴下面的不安全代码。

以下示例展示了不安全的DQL用法：

.. code-block:: php

    // INSECURE
    $dql = "SELECT u
              FROM MyProject\Entity\User u
             WHERE u.status = '" .  $_GET['status'] . "'
         ORDER BY " . $_GET['orderField'] . " ASC";

对于Doctrine来说，绝对没有办法找出 ``$dql`` 的哪些部分来自用户输入，哪些不是。
即使我们有自己的解析过程，这在技术上也是不可能的。正确的方法是：

.. code-block:: php

    $orderFieldWhitelist = array('email', 'username');
    $orderField = "email";

    if (in_array($_GET['orderField'], $orderFieldWhitelist)) {
        $orderField = $_GET['orderField'];
    }

    $dql = "SELECT u
              FROM MyProject\Entity\User u
             WHERE u.status = ?1
         ORDER BY u." . $orderField . " ASC";

    $query = $entityManager->createQuery($dql);
    $query->setParameter(1, $_GET['status']);

Preventing Mass Assignment Vulnerabilities
防止批量分配漏洞
------------------------------------------

ORM对CRUD应用非常方便，Doctrine也不例外。
但是，当天真地实现CRUD应用时，通常容易受到批量分配安全问题的影响

Doctrine开箱就不容易受到这个问题的影响，但是当你添加 ``updateFromArray()`` 或 ``updateFromJson()`` 类型的方法到它们时，你可以轻松地使你的实体容易受到批量分配。易受攻击的实体可能如下所示：
Doctrine is not vulnerable to this problem out of the box, but you can easily
make your entities vulnerable to mass assignment when you add methods of
the kind ``updateFromArray()`` or ``updateFromJson()`` to them. A vulnerable
entity might look like this:

.. code-block:: php

    /**
     * @Entity
     */
    class InsecureEntity
    {
        /** @Id @Column(type="integer") @GeneratedValue */
        private $id;
        /** @Column */
        private $email;
        /** @Column(type="boolean") */
        private $isAdmin;

        public function fromArray(array $userInput)
        {
            foreach ($userInput as $key => $value) {
                $this->$key = $value;
            }
        }
    }

现在，当你将整个请求数据传递给这个方法时，攻击者可以利用攻击者将“isAdmin”标志设置为true，从而实现质量对齐的可能性：
Now the possiblity of mass-asignment exists on this entity and can
be exploitet by attackers to set the "isAdmin" flag to true on any
object when you pass the whole request data to this method like:

.. code-block:: php

    $entity = new InsecureEntity();
    $entity->fromArray($_POST);

    $entityManager->persist($entity);
    $entityManager->flush();

你可以轻松地在这个非常简单的示例中发现此问题。但是，与框架和表单库结合使用时，出现此问题可能并不那么明显。小心避免这种错误。
You can spot this problem in this very simple example easily. However
in combination with frameworks and form libraries it might not be
so obvious when this issue arises. Be careful to avoid this
kind of mistake.

如何解决这个问题？你应始终通过批量分配功能设置允许键的白名单。
How to fix this problem? You should always have a whitelist
of allowed key to set via mass assignment functions.

.. code-block:: php

    public function fromArray(array $userInput, $allowedFields = array())
    {
        foreach ($userInput as $key => $value) {
            if (in_array($key, $allowedFields)) {
                $this->$key = $value;
            }
        }
    }
