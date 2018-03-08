> 原文[https://redis.io/topics/data-types-intro](https://redis.io/topics/data-types-intro)

# An introduction to Redis data types and abstractions

# 关于Redis数据类型及其概念的介绍

_Redis is not a plain key-value store, it is actually a data structures server, supporting different kinds of values. What this means is that, while in traditional key-value stores you associated string keys to string values, in Redis the value is not limited to a simple string, but can also hold more complex data structures. The following is the list of all the data structures supported by Redis, which will be covered separately in this tutorial:_

Redis不仅仅是纯粹的key-value的数据仓库，实际上是一种包含数据结构的服务应用，可以支持不同类型的值。这就是意味着，传统的key-value仓库就只将字符串作为key、字符串作为value，并将它们进行关联，但上在Redis中，值的类型不只是局限于字符串，还可以是其他复杂的数据结构。下面的列表就是Redis所能支持的数据结构，这些内容都会单独的在这篇教程中讲到：

* _Binary-safe strings._
* _Lists: collections of string elements sorted according to the order of insertion. They are basically linked lists._
* _Sets: collections of unique, unsorted string elements._
* _Sorted sets: similar to Sets but where every string element is associated to a floating number value, called score. The elements are always taken sorted by their score, so unlike Sets it is possible to retrieve a range of elements (for example you may ask: give me the top 10, or the bottom 10)._
* _Hashes: which are maps composed of fields associated with values. Both the field and the value are strings. This is very similar to Ruby or Python hashes._
* _Bit arrays (or simply bitmaps): it is possible, using special commands, to handle String values like an array of bits: you can set and clear individual bits, count all the bits set to 1, find the first set or unset bit, and so forth._
* _HyperLogLogs: this is a probabilistic data structure which is used in order to estimate the cardinality of a set. Don't be scared, it is simpler than it seems... See later in the HyperLogLog section of this tutorial._

* 字符串 Safe string：二进制安全的字符串
* 列表 List：字符串元素的集合，其顺序按照插入时的顺序
* 集合 Sets：不同元素的集合，其中的元素是唯一的，未排序的
* 有序集合 Sorted sets：与集合相似，但其中字符串元素都与一个浮点数值相关联，该浮点数称为`权值`。这些元素都是根据其权值进行排序。所以与Set不同之处在于`Sorted sets`可以得到一个范围内的元素(比如你可能这么要求：给我前面10个元素，或最后10个元素)
* 哈希 Hash：字段与值的映射表，其中字段与值都是字符串，和`Ruby`或`Python`中的哈希Hash类似
* 位数组 Bit arrays(简单来讲就是位图)：有可能的话，你会使用到特殊的命令，像位数组的形式来处理字符串：你可以设置和清除每一位的值，对每一位进行计数，取第一位或不设置这一位等等
* HyperLogLogs：这是一种用于基数的概率统计的数据结构，请不要害怕，它比看上去的要简单的多，在文档的后面章节中有关于`HyperLogLogs`的介绍

_It's not always trivial to grasp how these data types work and what to use in order to solve a given problem from the command reference, so this document is a crash course to Redis data types and their most common patterns._

对于数据类型如何工作，其实不需要经常去重要，所以显得不是很重要，然而对于一个指定的问题，怎么样才能将其解决，正因如此，这个文档可以作为一个紧急的知识课程，其中包含`Redis`数据结构的介绍，还有一些其常用的模式。

_For all the examples we'll use the redis-cli utility, a simple but handy command-line utility, to issue commands against the Redis server._

对于所有的示例，我们使用`redis-cli`工具进行操作，`redis-cli`工具一种简单的但随手可得的命令行工具，可用于与Redis服务进行命令通信。

## Redis keys

_Redis keys are binary safe, this means that you can use any binary sequence as a key, from a string like "foo" to the content of a JPEG file. The empty string is also a valid key._

Redis的key是二进制安全的，这就意味着你可以拿任何二进制序列作为key，比如拿字符串`foo`可以和一个`JPEG`文件的内容关联起来。空的字符串同样是一个有效的key。

_A few other rules about keys:_

其他一些关于key的规则：

* Very long keys are not a good idea. For instance a key of 1024 bytes is a bad idea not only memory-wise, but also because the lookup of the key in the dataset may require several costly key-comparisons. Even when the task at hand is to match the existence of a large value, hashing it (for example with SHA1) is a better idea, especially from the perspective of memory and bandwidth.
* Very short keys are often not a good idea. There is little point in writing "u1000flw" as a key if you can instead write "user:1000:followers". The latter is more readable and the added space is minor compared to the space used by the key object itself and the value object. While short keys will obviously consume a bit less memory, your job is to find the right balance.
* Try to stick with a schema. For instance "object-type:id" is a good idea, as in "user:1000". Dots or dashes are often used for multi-word fields, as in "comment:1234:reply.to" or "comment:1234:reply-to".
* The maximum allowed key size is 512 MB.

* 使用较长的key不是一个好主意。比如，一个长度为1024字节的是一个不好的主意，不仅仅是会存在内存溢出，还会在查找key的过程中需要较多次字符的比较，这是不小的代价。

