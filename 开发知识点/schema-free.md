# Schema-free

`Scheme-free`，中文翻译为模式自由。不同于传统的关系型数据库，使用`Schema-free database`存储数据，不需要提前定义其数据结构。

比如MySQL，在你向`MySQL`插入数据前，你需要提前定义好`table`的结构，并且插入的数据要与`table`中的列相对应。当然，你可以设置其中个别列为`NULL`，但他也是进行了赋值。

如`Scheme-free`的数据库`MongoDB`，在没有定义数据结构的情况下，可以直接向数据库插入数据。在不同数据结构的集合中，又可以进行聚合类操作。

下面介绍下`MongoDB`的特点：

* 基于文档数据模型
* 使用文档查询语言，支持动态查询
* 不需要模式迁移，(不同于关系型数据库，迁移必须要先同步数据库的表型结构)
* 便于水平扩展

在什么场景下使用：

* 预期未来你的应用拥有不同各类的数据结构
* 主要的业务逻辑需要通过编写数据库脚本语句


## 参考

[https://www.quora.com/What-is-schema-free-DB](https://www.quora.com/What-is-schema-free-DB)
[https://stackoverflow.com/questions/2117372/what-are-the-advantages-of-using-a-schema-free-database-like-mongodb-compared-to](https://stackoverflow.com/questions/2117372/what-are-the-advantages-of-using-a-schema-free-database-like-mongodb-compared-to)
