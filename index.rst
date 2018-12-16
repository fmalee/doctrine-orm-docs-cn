欢迎使用Doctrine 2 ORM文档！
==========================================

Doctrine文档包括教程、参考部分和解释对象关系映射器的不同部分的指南。

Doctrine DBAL and Doctrine Common 都有自己的文档。

获得帮助
------------

如果将本文档没有帮助回答你有关于Doctrine ORM的问题。你可以从不同的来源获得帮助：

-  解答常见问题的 :doc:`FAQ <reference/faq>`。
-  `Doctrine邮件列表 <http://groups.google.com/group/doctrine-user>`_
-  Freenode上的互联网中继聊天（IRC）: #doctrine
-  在 `JIRA <http://www.doctrine-project.org/jira>`_ 报告错误。
-  在 `Twitter <https://twitter.com/search/%23doctrine2>`_ 上使用 ``#doctrine2``
-  在 `StackOverflow <http://stackoverflow.com/questions/tagged/doctrine2>`_ 上

如果你需要更多不同的主题的架构，你可以浏览 :doc:`目录 <toc>`。

入门
---------------

* **教程**:
  :doc:`开始使用Doctrine <tutorials/getting-started>`

* **设置**:
  :doc:`安装 & 配置 <reference/configuration>`

映射对象到数据库
-------------------------------

* **映射**:
  :doc:`对象 <reference/basic-mapping>` |
  :doc:`关联 <reference/association-mapping>` |
  :doc:`继承 <reference/inheritance-mapping>`

* **驱动**:
  :doc:`文档注释 <reference/annotations-reference>` |
  :doc:`XML <reference/xml-mapping>` |
  :doc:`YAML <reference/yaml-mapping>` |
  :doc:`PHP <reference/php-mapping>`

使用对象
--------------------

* **基础参考**:
  :doc:`实体 <reference/working-with-objects>` |
  :doc:`关联 <reference/working-with-associations>` |
  :doc:`事件 <reference/events>`

* **快速参考**:
  :doc:`DQL <reference/dql-doctrine-query-language>` |
  :doc:`QueryBuilder <reference/query-builder>` |
  :doc:`原生SQL <reference/native-sql>`

* **内部**:
  :doc:`内部原理 <reference/unitofwork>` |
  :doc:`关联 <reference/unitofwork-associations>`

高级主题
---------------

* :doc:`架构 <reference/architecture>`
* :doc:`高级配置 <reference/advanced-configuration>`
* :doc:`限制和已知问题 <reference/limitations-and-known-issues>`
* :doc:`命令行工具 <reference/tools>`
* :doc:`事务和并发 <reference/transactions-and-concurrency>`
* :doc:`过滤器 <reference/filters>`
* :doc:`命名策略 <reference/namingstrategy>`
* :doc:`提升性能 <reference/improving-performance>`
* :doc:`高速缓存 <reference/caching>`
* :doc:`局部对象 <reference/partial-objects>`
* :doc:`变更跟踪策略 <reference/change-tracking-policies>`
* :doc:`最佳实践 <reference/best-practices>`
* :doc:`元数据驱动 <reference/metadata-drivers>`
* :doc:`批处理 <reference/batch-processing>`
* :doc:`二级缓存 <reference/second-level-cache>`

教程
---------

* :doc:`关联关系索引 <tutorials/working-with-indexed-associations>`
* :doc:`额外的延迟关联关系 <tutorials/extra-lazy-associations>`
* :doc:`复合主键 <tutorials/composite-primary-keys>`
* :doc:`有序关联 <tutorials/ordered-associations>`
* :doc:`分页 <tutorials/pagination>`
* :doc:`重写子类的字段/关联映射 <tutorials/override-field-association-mappings-in-subclasses>`
* :doc:`Embeddables <tutorials/embeddables>`

更新记录
----------

* :doc:`迁移至2.5 <changelog/migration_2_5>`

主题
--------

* **模式**:
  :doc:`聚合字段 <cookbook/aggregate-fields>` |
  :doc:`装饰模式 <cookbook/decorator-pattern>` |
  :doc:`策略模式 <cookbook/strategy-cookbook-introduction>`

* **DQL扩展点**:
  :doc:`DQL自定义Walkers <cookbook/dql-custom-walkers>` |
  :doc:`DQL用户定义函数 <cookbook/dql-user-defined-functions>`

* **实现**:
  :doc:`阵列访问 <cookbook/implementing-arrayaccess-for-domain-objects>` |
  :doc:`Notify ChangeTracking 示例 <cookbook/implementing-the-notify-changetracking-policy>` |
  :doc:`使用唤醒或克隆 <cookbook/implementing-wakeup-or-clone>` |
  :doc:`使用DateTime <cookbook/working-with-datetime>` |
  :doc:`验证 <cookbook/validation-of-entities>` |
  :doc:`会话中的实体 <cookbook/entities-in-session>` |
  :doc:`保持模块独立 <cookbook/resolve-target-entity-listener>`

* **集成至框架/库**
  :doc:`CodeIgniter <cookbook/integrating-with-codeigniter>`

* **隐藏的宝石**
  :doc:`表名称前缀 <cookbook/sql-table-prefixes>`

* **自定义数据类型**
  :doc:`MySQL枚举 <cookbook/mysql-enums>`
  :doc:`高级字段值转换 <cookbook/advanced-field-value-conversion-using-custom-mapping-types>`

.. include:: toc.rst
