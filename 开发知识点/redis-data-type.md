> 原文[https://redis.io/topics/data-types-intro](https://redis.io/topics/data-types-intro)

<!--
 _An introduction to Redis data types and abstractions_
-->

# 关于Redis数据类型及其概念的介绍

<!--
_Redis is not a plain key-value store, it is actually a data structures server, supporting different kinds of values. What this means is that, while in traditional key-value stores you associated string keys to string values, in Redis the value is not limited to a simple string, but can also hold more complex data structures. The following is the list of all the data structures supported by Redis, which will be covered separately in this tutorial:_
-->
Redis不仅仅是纯粹的key-value的数据仓库，实际上是一种包含数据结构的服务应用，可以支持不同类型的值。这就是意味着，传统的key-value仓库就只将字符串作为key、字符串作为value，并将它们进行关联，但上在Redis中，值的类型不只是局限于字符串，还可以是其他复杂的数据结构。下面的列表就是Redis所能支持的数据结构，这些内容都会单独的在这篇教程中讲到：

<!--
* _Binary-safe strings._
* _Lists: collections of string elements sorted according to the order of insertion. They are basically linked lists._
* _Sets: collections of unique, unsorted string elements._
* _Sorted sets: similar to Sets but where every string element is associated to a floating number value, called score. The elements are always taken sorted by their score, so unlike Sets it is possible to retrieve a range of elements (for example you may ask: give me the top 10, or the bottom 10)._
* _Hashes: which are maps composed of fields associated with values. Both the field and the value are strings. This is very similar to Ruby or Python hashes._
* _Bit arrays (or simply bitmaps): it is possible, using special commands, to handle String values like an array of bits: you can set and clear individual bits, count all the bits set to 1, find the first set or unset bit, and so forth._
* _HyperLogLogs: this is a probabilistic data structure which is used in order to estimate the cardinality of a set. Don't be scared, it is simpler than it seems... See later in the HyperLogLog section of this tutorial._
-->

* 字符串 Safe string：二进制安全的字符串
* 列表 List：字符串元素的集合，其顺序按照插入时的顺序
* 集合 Sets：不同元素的集合，其中的元素是唯一的，未排序的
* 有序集合 Sorted sets：与集合相似，但其中字符串元素都与一个浮点数值相关联，该浮点数称为`权值`。这些元素都是根据其权值进行排序。所以与Set不同之处在于`Sorted sets`可以得到一个范围内的元素(比如你可能这么要求：给我前面10个元素，或最后10个元素)
* 哈希 Hash：字段与值的映射表，其中字段与值都是字符串，和`Ruby`或`Python`中的哈希Hash类似
* 位数组 Bit arrays(简单来讲就是位图)：有可能的话，你会使用到特殊的命令，像位数组的形式来处理字符串：你可以设置和清除每一位的值，对每一位进行计数，取第一位或不设置这一位等等
* HyperLogLogs：这是一种用于基数的概率统计的数据结构，请不要害怕，它比看上去的要简单的多，在文档的后面章节中有关于`HyperLogLogs`的介绍

<!--
_It's not always trivial to grasp how these data types work and what to use in order to solve a given problem from the command reference, so this document is a crash course to Redis data types and their most common patterns._
-->
对于数据类型如何工作，其实不需要经常去重要，所以显得不是很重要，然而对于一个指定的问题，怎么样才能将其解决，正因如此，这个文档可以作为一个紧急的知识课程，其中包含`Redis`数据结构的介绍，还有一些其常用的模式。

<!--
_For all the examples we'll use the redis-cli utility, a simple but handy command-line utility, to issue commands against the Redis server._
-->
对于所有的示例，我们使用`redis-cli`工具进行操作，`redis-cli`工具一种简单的但随手可得的命令行工具，可用于与Redis服务进行命令通信。

## Redis keys

<!--
_Redis keys are binary safe, this means that you can use any binary sequence as a key, from a string like "foo" to the content of a JPEG file. The empty string is also a valid key._
-->
Redis的key是二进制安全的，这就意味着你可以拿任何二进制序列作为key，比如拿字符串`foo`可以和一个`JPEG`文件的内容关联起来。空的字符串同样是一个有效的key。

<!--
_A few other rules about keys:_
-->
其他一些关于key的规则：

<!--
* _Very long keys are not a good idea. For instance a key of 1024 bytes is a bad idea not only memory-wise, but also because the lookup of the key in the dataset may require several costly key-comparisons. Even when the task at hand is to match the existence of a large value, hashing it (for example with SHA1) is a better idea, especially from the perspective of memory and bandwidth._
* _Very short keys are often not a good idea. There is little point in writing "u1000flw" as a key if you can instead write "user:1000:followers". The latter is more readable and the added space is minor compared to the space used by the key object itself and the value object. While short keys will obviously consume a bit less memory, your job is to find the right balance._
* _Try to stick with a schema. For instance "object-type:id" is a good idea, as in "user:1000". Dots or dashes are often used for multi-word fields, as in "comment:1234:reply.to" or "comment:1234:reply-to"._
* _The maximum allowed key size is 512 MB._
-->

* 使用较长的key不是一个好主意。比如，一个长度为1024字节的是一个不好的主意，不仅仅是会存在内存溢出的情况，还因为在查找key的过程中进行多次字符的比较，这是不小的代价。当手头上的工作是匹配一个较长的值时，特别是从内存和值位宽的角度来说，将该值进行哈希(比如使用`SHA1`)是一个不错的主意。
* 非常短的key也不是一个好的主意。其中有一点值得说的是，如果你可以使用`user:1000:followers`代替`u1000flw`作为key。代替之后的key更具有可读性，而且与key对象本身和value对象所占空间相比，增加的空间就显得比较不重要了。但是短点的key很显然占用较少的内存，你的任务就是要找到一个两者的平衡点。
* 尽量将对象类型在key中进行体现。比如`object-type:id`就是一个不错的主意，像`user:1000`这样。点`.`和连接符`-`常常用于有个单词组成的字段，比如`comment:1234:reply.to`或`comment:1234:reply-to`
* 最大允许的key的大小是`512MB`
<!--
 _Redis Strings_
-->
## Redis 字符串

<!--
_The Redis String type is the simplest type of value you can associate with a Redis key. It is the only data type in Memcached, so it is also very natural for newcomers to use it in Redis._
-->
你能想到可以作为Redis的key值中，字符串类型是最简单的。这也是唯一在内存缓存的数据类型，所以对在Redis上使用该类型的新手来说也是非常自然的。

<!--
_Since Redis keys are strings, when we use the string type as a value too, we are mapping a string to another string. The string data type is useful for a number of use cases, like caching HTML fragments or pages._
-->
因为Redis的key是字符串类型，所以我们也用字符串来表示value，我们可以将一个字符串映射到另一个字符串。字符串作为value，在许多使用案例中都非常有用，如可以用于缓存HTML片段或页面。

<!--
_Let's play a bit with the string type, using redis-cli (all the examples will be performed via redis-cli in this tutorial)._
-->
通过使用`redis-cli`(教程中的所有示例通过`redis-cli`被执行)，让我们来尝试一下字符串类型。

```redis
> set mykey somevalue
OK
> get mykey
"somevalue"
```
<!--
_As you can see using the SET and the GET commands are the way we set and retrieve a string value. Note that SET will replace any existing value already stored into the key, in the case that the key already exists, even if the key is associated with a non-string value. So SET performs an assignment._
-->
如你所见，使用`SET`和`GET`命令是我们设置和获取一个字符串值的方法。值得注意的是，`SET`会替换任何已经存在，且保存于该key中的值，在这种情况下，都是表示已经存在了的key，即使这个key映射到的是一个非字符串的值。所以`SET`命令执行的是一个分配的任务。
<!--
_Values can be strings (including binary data) of every kind, for instance you can store a jpeg image inside a value. A value can't be bigger than 512 MB._
-->
value可以是表示各个类型的字符串(包括二进制数据)，比如你可以在value中保存一张`jpeg`的图片。value的大小不能超过512MB。
<!--
_The SET command has interesting options, that are provided as additional arguments. For example, I may ask SET to fail if the key already exists, or the opposite, that it only succeed if the key already exists:_
-->
`SET`命令也有一些有趣的选项，并以歌德参数的形式给出。比如，你可以希望当key值存在时不进行设置，或者反过来，当key存在时才进行设置。

```redis
> set mykey newval nx
(nil)
> set mykey newval xx
OK
```
<!--
_Even if strings are the basic values of Redis, there are interesting operations you can perform with them. For instance, one is atomic increment:_
-->
即使字符串在Redis中作为最基本的value类型，但你可以用它来执行一些有意思的操作。比如，原子性的自增操作。

```redis
> set counter 100
OK
> incr counter
(integer) 101
> incr counter
(integer) 102
> incrby counter 50
(integer) 152
```

<!--
_The `INCR command parses the string value as an integer, increments it by one, and finally sets the obtained value as the new value. There are other similar commands like INCRBY, DECR and DECRBY. Internally it's always the same command, acting in a slightly different way._
-->

`INCR`命令将字符串值解析成一个整型，并自增加1，最终将得到的值设置为新的值。还有其他类似的命令，如`INCRBY`，`DECR`，`DECRBY`。在内部实现看来其实是相同的命令，但在执行过程中有微微的不同。

<!--
_What does it mean that INCR is atomic? That even multiple clients issuing INCR against the same key will never enter into a race condition. For instance, it will never happen that client 1 reads "10", client 2 reads "10" at the same time, both increment to 11, and set the new value to 11. The final value will always be 12 and the read-increment-set operation is performed while all the other clients are not executing a command at the same time._
-->

为什么说`INCR`操作是原子性的？那就是即使有多个客户端对同一个key进行自增操作，并不会造成拥堵的状态。比如，当两个客户端在同一时刻将同一个值自增到`11`，那么两个客户端同时读到`10`是不会发生的。最终的值总会是`12`，只有在相同的时刻，在没有其他执行命令的情况下，才会执行`读取-自增-设置`操作。

<!--
_There are a number of commands for operating on strings. For example the GETSET command sets a key to a new value, returning the old value as the result. You can use this command, for example, if you have a system that increments a Redis key using INCR every time your web site receives a new visitor. You may want to collect this information once every hour, without losing a single increment. You can GETSET the key, assigning it the new value of "0" and reading the old value back._
-->

有很多关于字符串的命令。比如`GETSET`，设置一个新的值，并将旧的值作为结果返回。如果你的系统在每一时刻需要使用`INCR`用于表示又有一个新的访问者。这类信息你可以要每小时收集一次，并且不要丢掉一个访问者数。你可以使用`GETSET`，将`0`作为最新的值，并将读取旧的值。

<!--
_The ability to set or retrieve the value of multiple keys in a single command is also useful for reduced latency. For this reason there are the MSET and MGET commands:_
-->

为了减少延迟，可以在单一个命令中同时设置或读取多个key，这个能力相当有用。基于这个原因，就有了`MSET`和`MGET`：

```redis
> mset a 10 b 20 c 30
OK
> mget a b c
1) "10"
2) "20"
3) "30"
```

<!--
_When MGET is used, Redis returns an array of values._
-->

当使用`MGET`时，会返回一个数组。

<!--
 Altering and querying the key space
-->

## key的修改和查询

<!--
_There are commands that are not defined on particular types, but are useful in order to interact with the space of keys, and thus, can be used with keys of any type._
-->

还有一些不是定义在特定类型上的命令，在于key的交互中非常有用，因此，不管该key值映射到何种类型的值，都可以使用。

<!--
_For example the EXISTS command returns 1 or 0 to signal if a given key exists or not in the database, while the DEL command deletes a key and associated value, whatever the value is._
-->

比如，可以通过`EXISTS`命令来查询给定的key是否存在于数据库中，返回`1`表示有，返回`0`表示没有，同时，使用`DEL`命令可以删除一个key并删除与之关联的value，而且不需要在乎是什么值。

```redis
> set mykey hello
OK
> exists mykey
(integer) 1
> del mykey
(integer) 1
> exists mykey
(integer) 0
```

<!--
_From the examples you can also see how DEL itself returns 1 or 0 depending on whether the key was removed (it existed) or not (there was no such key with that name)._
-->

从上述的例子中可以看出，执行`DEL`操作会根据当前key是否存在，如果存在则被删除返回1，如果不存在则返回0。

<!--
_There are many key space related commands, but the above two are the essential ones together with the TYPE command, which returns the kind of value stored at the specified key:_
-->

还有很关于key操作的命令，但上述的两个命令本质上都是由`TYPE`命令实现的，对于特定的key，返回该key保存的value的类型：

```redis
> set mykey x
OK
> type mykey
string
> del mykey
(integer) 1
> type mykey
none
```

<!--
 _Redis expires: keys with limited time to live_
-->

## key的过期时间

<!--
_Before continuing with more complex data structures, we need to discuss another feature which works regardless of the value type, and is called Redis expires. Basically you can set a timeout for a key, which is a limited time to live. When the time to live elapses, the key is automatically destroyed, exactly as if the user called the DEL command with the key._
-->

在继续处理更复杂的数据类型前，我们需要来讨论另一种特性，这个特性适用于任何类型，叫做`过期时间`。最基本的事，你可以为一个key设置一个过期时间，这个key只在限定的时间内有效。当超过这个限定时间内，就会销毁这个key，就好像用户对这个key执行了`DEL`操作一样。

<!--
_A few quick info about Redis expires:_
-->

快速了解下关于`过期时间`的知识：

<!--
* They can be set both using seconds or milliseconds precision.
* However the expire time resolution is always 1 millisecond.
* Information about expires are replicated and persisted on disk, the time virtually passes when your Redis server remains stopped (this means that Redis saves the date at which a key will expire).
-->

* 在精确度上，可以设置为秒级别或毫秒级别
* 然而过期时间总是`1毫秒`
* 过期时间的信息是可再生的保存在磁盘上，即使`Redis`服务器停止了，也是按真实的时间来计算(这就意味着Redis上保存了这个key将要过期的时间)

<!--
_Setting an expire is trivial:_
-->

设置一个过期时间非常简单：

```redis
> set key some-value
OK
> expire key 5
(integer) 1
> get key (immediately)
"some-value"
> get key (after some time)
(nil)
```

<!--
_The key vanished between the two GET calls, since the second call was delayed more than 5 seconds. In the example above we used EXPIRE in order to set the expire (it can also be used in order to set a different expire to a key already having one, like PERSIST can be used in order to remove the expire and make the key persistent forever). However we can also create keys with expires using other Redis commands. For example using SET options:_
-->

因为两调用`GET`时间间隔大于5，所以`key`已经消失了。在上述的例子中使用`EXPIRE`来设置一个过期时间(这个方法也同样适用于为已经有过期时间的key设置一个新的过期时间)，就好像使用`PERSIST`来移除过期时间，让其永远都存在。然而，我们同样可以使用别的`Redis`命令来创建带有过期时间的key。比如使用`SET`命令选项：

```redis
> set key 100 ex 10
OK
> ttl key
(integer) 9
```

<!--
_The example above sets a key with the string value 100, having an expire of ten seconds. Later the TTL command is called in order to check the remaining time to live for the key._
-->

这述的例子中，我们创建一个key，这个key保存了一个字符串值`100`，并且有`10`秒的过期时间。接下来，使用`TTL`命令来检查这个key还能保存多长时间。

<!--
_In order to set and check expires in milliseconds, check the PEXPIRE and the PTTL commands, and the full list of SET options._
-->

为了以毫秒的形式来设置和检查某个key，可以使用`PEXPIRE`和`PTTL`。也可以使用`SET`命令中的`px`选项。

<!--
 _Redis Lists_
-->

## Redis List

<!--
_To explain the List data type it's better to start with a little bit of theory, as the term List is often used in an improper way by information technology folks. For instance "Python Lists" are not what the name may suggest (Linked Lists), but rather Arrays (the same data type is called Array in Ruby actually)._
-->

因为在不同的技术分支中，`List`经常以一种不恰当的方式使用，为了解释`List`这种数据类型，最好还是从一点理论知识开始讲起。比如，在`Python Lists`并不像他名字提示的这样(链式线性表)，他其实是一个数组(在`Ruby`中，与之相同的数据类型确实叫做数组)。

<!--
_From a very general point of view a List is just a sequence of ordered elements: 10,20,1,2,3 is a list. But the properties of a List implemented using an Array are very different from the properties of a List implemented using a Linked List._
-->

从一个常规的视角来看，一个`List`就是一系列有序的序列：10,20,1,2,3，就是一个`List`。但是使用数组实现的`List`和使用链式线性表表实现的`List`有很大的不同之处。

<!--
_Redis lists are implemented via Linked Lists. This means that even if you have millions of elements inside a list, the operation of adding a new element in the head or in the tail of the list is performed in constant time. The speed of adding a new element with the LPUSH command to the head of a list with ten elements is the same as adding an element to the head of list with 10 million elements._
-->

`Redis`中的`List`是通过链式线性表，这就意味着，即使在你的列表中有多如百万计的元素，在首尾新增一个新元素的操作也能马上执行。使用`LPUSH`命令对一个只有10个元素的`List`和向一个有1000万元素的列表进行操作，在他们的首部插入一个新的元素，他们的速度是一样的。

<!--
_What's the downside? Accessing an element by index is very fast in lists implemented with an Array (constant time indexed access) and not so fast in lists implemented by linked lists (where the operation requires an amount of work proportional to the index of the accessed element)._
-->

那有什么不好的特性吗？使用数组实现的`List`，通过索引方式访问是非常快的，但在以链式线性表实现的`List`中就没有那么快(该操作需要大量的工作来指定元素，工作规模与`List`的长度成正比)

> 数组具有随机存取的优势，链式表首尾操作，时间复杂度为`O(1)`

<!--
_Redis Lists are implemented with linked lists because for a database system it is crucial to be able to add elements to a very long list in a very fast way. Another strong advantage, as you'll see in a moment, is that Redis Lists can be taken at constant length in constant time._
-->

`Redis List`是使用链式线性表来实现的，因为对于一个数据库系统来说，其最为关键的能力是，在尽可能快的时间内，在非常长的列表中新增一个元素。一会儿你会看这种方式的其他优势，那就是`Redis List`可以在常数阶的时间内得到常数阶的长度。

<!--
_When fast access to the middle of a large collection of elements is important, there is a different data structure that can be used, called sorted sets. Sorted sets will be covered later in this tutorial._
-->

当你的应用中，更看重如何在一个较大集合中更快地获取其中间的元素，也有一个不同的数据结构`有序List`可以使用。`有序List`会在后面的教程中提及。

<!--
First steps with Redis Lists
-->

## 走出第一步

<!--
_The LPUSH command adds a new element into a list, on the left (at the head), while the RPUSH command adds a new element into a list ,on the right (at the tail). Finally the LRANGE command extracts ranges of elements from lists:_
-->

`LPUSH`命令从左边(首部)，向一个`list`新加一个元素，同时，`RPUSH`命令从右边(尾部)，向一个`list`新加一个元素。最后使用`LRANGE`命令可以从`list`中取得一个区间内的元素。

```redis
> rpush mylist A
(integer) 1
> rpush mylist B
(integer) 2
> lpush mylist first
(integer) 3
> lrange mylist 0 -1
1) "first"
2) "A"
3) "B"
```

<!--
_Note that LRANGE takes two indexes, the first and the last element of the range to return. Both the indexes can be negative, telling Redis to start counting from the end: so -1 is the last element, -2 is the penultimate element of the list, and so forth._
-->

注意，`LRANGE`命令使用两个下标，分别表示第一个元素和第二个元素在在列表中的下标。这个两个下标都是可以为负数，这样就告诉`Redis`从尾部开始计算：所以`-1`表示最后一个元素，`-2`表示倒数第二个元素，以此类推。

<!--
_As you can see RPUSH appended the elements on the right of the list, while the final LPUSH appended the element on the left._
-->

正如你所看到的一样，`RPUSH`将元素新增于`list`的右边，同时，最后一个`LPUSH`将元素新增于`list`的左边。

<!--
_Both commands are variadic commands, meaning that you are free to push multiple elements into a list in a single call:_
-->

这两个命令都可变参数的命令，这就意味着，在一次的调用中，你完全可以在一个`list`中插入多个元素。

```redis
> rpush mylist 1 2 3 4 5 "foo bar"
(integer) 9
> lrange mylist 0 -1
1) "first"
2) "A"
3) "B"
4) "1"
5) "2"
6) "3"
7) "4"
8) "5"
9) "foo bar"
```

<!--
_An important operation defined on Redis lists is the ability to pop elements. Popping elements is the operation of both retrieving the element from the list, and eliminating it from the list, at the same time. You can pop elements from left and right, similarly to how you can push elements in both sides of the list:_
-->

还有一个重要的操作可以操作`List`，这个操作的能力就是可以`pop`元素。`pop`可以同时从列表中获取一个元素，并将该元素从这个列表中移除。你可以从列表的左边或者右边`pop`元素，和你怎么在列表左右新加元素是一样的：

```redis
> rpush mylist a b c
(integer) 3
> rpop mylist
"c"
> rpop mylist
"b"
> rpop mylist
"a"
```

<!--
_We added three elements and popped three elements, so at the end of this sequence of commands the list is empty and there are no more elements to pop. If we try to pop yet another element, this is the result we get:_
-->

你向一个列表中新加3个元素，随后又移除了3个元素，所以执行这一系列命令后，这个列表是空的，列表中不存在任何元素。如果你有去执行`pop`操作，你会得到下面的结果：

```redis
> rpop mylist
(nil)
```

<!--
_Redis returned a NULL value to signal that there are no elements in the list._
-->

Redis返回一个`NULL`值，用于标识在这个列表中已经没有元素了。

<!--
 _Common use cases for lists_
-->

## Lists的常用用法

<!--
_Lists are useful for a number of tasks, two very representative use cases are the following:_
-->

列表应用于很多的任务当中，非常有用。下面有两个非常有代表性的示例：

<!--
* _Remember the latest updates posted by users into a social network._
* _Communication between processes, using a consumer-producer pattern where the producer pushes items into a list, and a consumer (usually a worker) consumes those items and executed actions. Redis has special list commands to make this use case both more reliable and efficient._
-->

* 在社交网站上，记得用户最近更新、发表的内容
* 进程之间进行通信，使用`消费者-生产者`的模式时，当一个生产者向列表中新增一些元素，另一方面有一个消费者(通常是woker)去处理这些元素，并执行相应的动作。`Redis`有特殊的列表命令，可以保证在这种使用场景下，可靠性更强，效率更高。

<!--
_For example both the popular Ruby libraries resque and sidekiq use Redis lists under the hood in order to implement background jobs._
-->

如，有两个非常有名的`Ruby`库，`resque`和`sidekiq`，都在后台使用`Redis`的列表来实现后台任务。

<!--
_The popular Twitter social network takes the latest tweets posted by users into Redis lists._
-->

著名的社交网站`Twitter`使用`Redis`列表，来实现提取最近更新的内容。

<!--
_To describe a common use case step by step, imagine your home page shows the latest photos published in a photo sharing social network and you want to speedup access._
-->

为了一步一步描述常用用法，想像一下，你的社交主页展示了最近一次发表于照片圈的一张图片，然后你希望让访问速度变快。

<!--
* _Every time a user posts a new photo, we add its ID into a list with LPUSH._
* _When users visit the home page, we use LRANGE 0 9 in order to get the latest 10 posted items._
-->

* 每一次用户发表一张新的图片，我们使用`LPUSH`将该图片的ID加入到列表中
* 当用户访问主页时，我们使用`LRANGE 0 9`，获得最近更新的10张图片

<!--
Capped lists
-->

## 伪固定长度列表

<!--
_In many use cases we just want to use lists to store the latest items, whatever they are: social network updates, logs, or anything else._
-->

在多数的情况下，我们只是想用列表来保存最近更新的内容，不管他们是什么，社交网站的更新内容也好，日志或者其他什么东西也好，都可以。

<!--
_Redis allows us to use lists as a capped collection, only remembering the latest N items and discarding all the oldest items using the LTRIM command._
-->

`Redis`允许你用列表创建一个伪固定长度的集合，只用来保存更新的`N`个元素，并通过`LTRIM`命令移除所以时间最久的元素。

<!--
_The LTRIM command is similar to LRANGE, but instead of displaying the specified range of elements it sets this range as the new list value. All the elements outside the given range are removed._
-->

`LTRIM`命令与`LRANGE`有些相似，但与展示一个特定区间内的元素不同的是，他会将这个特定区间作为列表新的值。所有不在这个区间内的元素都会被移除。

<!--
_An example will make it more clear:_
-->

一个示例可以更好地说明：

```redis
> rpush mylist 1 2 3 4 5
(integer) 5
> ltrim mylist 0 2
OK
> lrange mylist 0 -1
1) "1"
2) "2"
3) "3"
```

<!--
_The above LTRIM command tells Redis to take just list elements from index 0 to 2, everything else will be discarded. This allows for a very simple but useful pattern: doing a List push operation + a List trim operation together in order to add a new element and discard elements exceeding a limit:_
-->

上述的`LTRIM`告诉`Redis`只需要从列表中取下标为`0`到`2`的元素，其他的元素都会被丢弃。让一个简单但有用的模式得以行得通：一个新增`push`元素操作加一个截取`trim`元素操作，当超过一定限制后，可以实现每新加一个元素就可以移除一个元素：

```redis
LPUSH mylist <some element>
LTRIM mylist 0 999
```

<!--
_The above combination adds a new element and takes only the 1000 newest elements into the list. With LRANGE you can access the top items without any need to remember very old data._
-->

上面的操作结合了新加一个元素和只取1000最新的元素。这样，你就可以使用`LRANGE`操作选取上面一些元素，而不需要去记住那些非常老的数据。

<!--
_Note: while LRANGE is technically an O(N) command, accessing small ranges towards the head or the tail of the list is a constant time operation._
-->

注意：`LRANGE`是一个时间复杂度为`O(N)`的命令，从首部还尾部访问一个区间的元素是一个固定时间的操作。

<!--
 Blocking operations on lists
-->

## List的锁操作

<!--
_Lists have a special feature that make them suitable to implement queues, and in general as a building block for inter process communication systems: blocking operations._
-->

列表有一个特殊的特性，使它们适合于实现队列，并且经常作为进程间通信系统的构造锁芯，那就是锁操作。

<!--
_Imagine you want to push items into a list with one process, and use a different process in order to actually do some kind of work with those items. This is the usual producer / consumer setup, and can be implemented in the following simple way:_
-->

想像一下，在其中一个进程中，你向列表新加一些元素，然后使用不同的进程，利用这些元素执行某些工作。这个就是很普通的`生产者/消费者`的模式，可以用下面简单的方法实现：

<!--
* _To push items into the list, producers call LPUSH._
* _To extract / process items from the list, consumers call RPOP._
-->

* 生产者使用`LPUSH`向列表新加数据
* 消费者使用`RPOP`来提取并处理列表中的数据

<!--
_However it is possible that sometimes the list is empty and there is nothing to process, so RPOP just returns NULL. In this case a consumer is forced to wait some time and retry again with RPOP. This is called polling, and is not a good idea in this context because it has several drawbacks:_
-->

然而，有可能在某些时刻，列表是空的，那就没有数据需要处理，所以`RPOP`会返回`NULL`。在这种情况下，消费者被强制等待一段时间，并不断重新使用`RPOP`来获取数据。这个就是叫做轮询，在执行环境中，这是种不好的方法，因为这种方式存在几个缺点：

<!--
* _Forces Redis and clients to process useless commands (all the requests when the list is empty will get no actual work done, they'll just return NULL)._
* _Adds a delay to the processing of items, since after a worker receives a NULL, it waits some time. To make the delay smaller, we could wait less between calls to RPOP, with the effect of amplifying problem number 1, i.e. more useless calls to Redis._
-->

* `Redis`和客户端强制执行了没用的命令(当列表为空的时候，所有的请求都返回NULL，没有做任何实际的工作)
* 给处理数据的进程增加一个延时时间，因为在一台`worker`收到`NULL`时，他需要等待一段时间。为了让延时时间短点，我们可以设置一个较短的延时时间，但带来的影响就如#1中描述的那样，即对`Redis`执行了更多无用的命令。

<!--
_So Redis implements commands called BRPOP and BLPOP which are versions of RPOP and LPOP able to block if the list is empty: they'll return to the caller only when a new element is added to the list, or when a user-specified timeout is reached._
-->

所以`Redis`实现了`BRPOP`和`BLPOP`两个命令，分别是`RPOP`和`LPOP`的变体，当列表为空的时间，可以用来阻塞进程。当列表中加入新的元素或过了用户设置的超时时间，才会给调用才返回信息。

<!--
_This is an example of a BRPOP call we could use in the worker:_
-->

下面是在`worker`机上调用`BRPOP`的示例：

```redis
> brpop tasks 5
1) "tasks"
2) "do_something"
```

<!--
_It means: "wait for elements in the list tasks, but return if after 5 seconds no element is available"._
-->

他的意思是：等待一个需要弹出的元素，但在5秒后还是没有有效元素就返回

<!--
_Note that you can use 0 as timeout to wait for elements forever, and you can also specify multiple lists and not just one, in order to wait on multiple lists at the same time, and get notified when the first list receives an element._
-->

注意，你可以设置超时时间为`0`，让这个任务一直存在，同时你可以指定多个列表，多个列表可同时等待，接收到一个元素后执行操作

<!--
_A few things to note about BRPOP:_
-->

关于`BRPOP`，有一些东西值得注意：

<!--
* _Clients are served in an ordered way: the first client that blocked waiting for a list, is served first when an element is pushed by some other client, and so forth._
* _The return value is different compared to RPOP: it is a two-element array since it also includes the name of the key, because BRPOP and BLPOP are able to block waiting for elements from multiple lists._
* _If the timeout is reached, NULL is returned._

_There are more things you should know about lists and blocking ops. We suggest that you read more on the following:_

* _It is possible to build safer queues or rotating queues using RPOPLPUSH._
* _There is also a blocking variant of the command, called BRPOPLPUSH._
-->

* 服务器有序地为客户端提供服务：第一个客户端会阻塞该列表，接到到元素后，第一个执行，并从列表中退出，以此类推。
* 其返回值与`RPOP`的返回值不同：返回值是两个元素的数组，其中包含一个key，那是因为`BRPOP`和`BLPOP`都可以阻塞，以等待一个元素
* 过了超时时间将返回NULL

关于列表和阻塞操作，你还需要知道其他一些知识。我们建议基于以下两点去了解更多：

* 可以使用`RPOPLPUSH`，创建更安全的队列或者循环队列
* BRPOPLPUSH命令也可以造成阻塞

<!--
 _Automatic creation and removal of keys_
-->

## key的自动创建与删除

<!--
_So far in our examples we never had to create empty lists before pushing elements, or removing empty lists when they no longer have elements inside. It is Redis' responsibility to delete keys when lists are left empty, or to create an empty list if the key does not exist and we are trying to add elements to it, for example, with LPUSH._
-->

目前为止，在我们的例子中，还没有创建过一个空的列表，当一个有元素的列表，将其元素全部删除，如何去删除这个遗留下来的空列表，我们都没有提及。处理遗留下来的空列表及创建要插入元素的空列表，`Redis`会自动帮我们完成。

<!--
This is not specific to lists, it applies to all the Redis data types composed of multiple elements Sets, Sorted Sets and Hashes.
-->

`Redis`不是只对列表做这些工作，再包含多个元素的数据类型里也两样适用，如集合、有序集合、哈希

<!--
_Basically we can summarize the behavior with three rules:_
-->

基本上，我们可以总结出以下三个规则：

<!--
* _When we add an element to an aggregate data type, if the target key does not exist, an empty aggregate data type is created before adding the element._
* _When we remove elements from an aggregate data type, if the value remains empty, the key is automatically destroyed._
* _Calling a read-only command such as LLEN (which returns the length of the list), or a write command removing elements, with an empty key, always produces the same result as if the key is holding an empty aggregate type of the type the command expects to find._
-->

* 当我们向聚合类数据类型中增加一个元素，如果目标的key不存在，那在插入之前会先创建一个空的聚合类数据类型
* 当我们从一个聚合类数据类型中移除元素，当其为空时，key会被自动销毁。
* 调用一个只读型的命令，如`LLEN`(返回列表的长度)，还有移除元素的写命令应用于空的key时，总是会产生相同的结构，就好像key对应的空的聚合类型总是能找到。

<!--
_Examples of rule 1:_
-->

规则1的示例：

```redis
> del mylist
(integer) 1
> lpush mylist 1 2 3
(integer) 3
```

<!--
_However we can't perform operations against the wrong type if the key exists:_
-->

但我们不能在错误的类型上执行这种操作

```redis
> set foo bar
OK
> lpush foo 1 2 3
(error) WRONGTYPE Operation against a key holding the wrong kind of value
> type foo
string
```

<!--
_Example of rule 2:_
-->

规则2的示例：

```redis
> lpush mylist 1 2 3
(integer) 3
> exists mylist
(integer) 1
> lpop mylist
"3"
> lpop mylist
"2"
> lpop mylist
"1"
> exists mylist
(integer) 0
```

<!--
_The key no longer exists after all the elements are popped._
-->

当所有的元素都已弹出，则key将不再存在。

<!--
_Example of rule 3:_
-->

规则3的示例：

```redis
> del mylist
(integer) 0
> llen mylist
(integer) 0
> lpop mylist
(nil)
```

<!--
 _Redis Hashes_
-->

## Redis 哈希类型

<!--
_Redis hashes look exactly how one might expect a "hash" to look, with field-value pairs:_
-->

Redis中的哈希类型看起来就好像一个包含键值对的哈希。

```redis
> hmset user:1000 username antirez birthyear 1977 verified 1
OK
> hget user:1000 username
"antirez"
> hget user:1000 birthyear
"1977"
> hgetall user:1000
1) "username"
2) "antirez"
3) "birthyear"
4) "1977"
5) "verified"
6) "1"
```

<!--
_While hashes are handy to represent objects, actually the number of fields you can put inside a hash has no practical limits (other than available memory), so you can use hashes in many different ways inside your application._
-->

哈希常常用于代表对象，且操作哈希时并没有实际的限制(除了内存不足)，所以在你的应用中，哈希的使用方法很多。

<!--
_The command HMSET sets multiple fields of the hash, while HGET retrieves a single field. HMGET is similar to HGET but returns an array of values:_
-->

`HMSET`设置一个包含多个字段的哈希，而`HGET`用于设置单个字段。`HMGET`和`HGET`相似，但以数组的形式返回

```redis
> hmget user:1000 username birthyear no-such-field
1) "antirez"
2) "1977"
3) (nil)
```

<!--
_There are commands that are able to perform operations on individual fields as well, like HINCRBY:_
-->

还有一些命令可在其中单个字段上执行，如`HINCRBY`

```redis
> hincrby user:1000 birthyear 10
(integer) 1987
> hincrby user:1000 birthyear 10
(integer) 1997
```

<!--
_You can find the full list of hash commands in the documentation._
-->

所有有关哈希的操作可在[文档](http://redis.io/commands#hash)中查看。

<!--
_It is worth noting that small hashes (i.e., a few elements with small values) are encoded in special way in memory that make them very memory efficient._
-->

值得做的是，将少量相关字段的元素编码成一个哈希，更方便于我们记忆。

## _Redis Sets_

## Reddis 集合

<!--
_Redis Sets are unordered collections of strings. The SADD command adds new elements to a set. It's also possible to do a number of other operations against sets like testing if a given element already exists, performing the intersection, union or difference between multiple sets, and so forth._
-->

Redis的集合是字符串的无序集合，使用`SADD`向一个集合中增加一个元素。当然还有好多应用于集合的命令及操作，如判断元素是否存在、集合类的运算等等。

```redis
> sadd myset 1 2 3
(integer) 3
> smembers myset
1. 3
2. 1
3. 2
```

<!--
Here I've added three elements to my set and told Redis to return all the elements. As you can see they are not sorted  Redis is free to return the elements in any order at every call, since there is no contract with the user about element ordering.
-->

这里我向集合中增加了三个元素，最后返回所有的元素。对于每一次的调用，其返回结果的顺序是不确定，因为在元素如何排序，我们并没有定义。

<!--
_Redis has commands to test for membership. For example, checking if an element exists:_
-->

`Redis`有命令可以测试元素之间的关系。比如，检查元素是否存在：

```redis
> sismember myset 3
(integer) 1
> sismember myset 30
(integer) 0
```

<!--
"3" is a member of the set, while "30" is not.

Sets are good for expressing relations between objects. For instance we can easily use sets in order to implement tags.

A simple way to model this problem is to have a set for every object we want to tag. The set contains the IDs of the tags associated with the object.

One illustration is tagging news articles. If article ID 1000 is tagged with tags 1, 2, 5 and 77, a set can associate these tag IDs with the news item:
-->

`3`是集合中的成员，`30`不是。

集合可以很好的表达对象间的关系。比如，可以使用集合还实现标签的功能。

只要有一个需要标记的对象集合就可以很简单地模拟这个问题。这个集合中包含与对象有关标签的`ID`

先以为新闻文章做标记作为示例。如果`ID`为1000的文章标记为`1,2,5,77`，再使用一个集合将新闻元素与标记的ID相关联。

```redis
> sadd news:1000:tags 1 2 5 77
(integer) 4
```

_We may also want to have the inverse relation as well: the list of all the news tagged with a given tag:_

同样，我们也可以将标记与文章之间的使用相反的关系：一个新闻的列表，且新闻已被标记

```redis
> sadd tag:1:news 1000
(integer) 1
> sadd tag:2:news 1000
(integer) 1
> sadd tag:5:news 1000
(integer) 1
> sadd tag:77:news 1000
(integer) 1
```

<!--
To get all the tags for a given object is trivial:

Note: in the example we assume you have another data structure, for example a Redis hash, which maps tag IDs to tag names.

There are other non trivial operations that are still easy to implement using the right Redis commands. For instance we may want a list of all the objects with the tags 1, 2, 10, and 27 together. We can do this using the SINTER command, which performs the intersection between different sets. We can use:
-->

得到一个对象的所有标签就非常简单：

```redis
> smembers news:1000:tags
1. 5
2. 1
3. 77
4. 2
```

注意：在这个例子中，我们已经假设你已将`ID`映射到标签名的数据结构，如哈希。

使用合适的命令，还可以实现其他一些集合操作。比如希望返回所有的标签，可以使用`SINTER`命令，用于在不同的集合中取交集。

```redis
> sinter tag:1:news tag:2:news tag:10:news tag:27:news
... results here ...
```

<!--
In addition to intersection you can also perform unions, difference, extract a random element, and so forth.

The command to extract an element is called SPOP, and is handy to model certain problems. For example in order to implement a web-based poker game, you may want to represent your deck with a set. Imagine we use a one-char prefix for (C)lubs, (D)iamonds, (H)earts, (S)pades:
-->

当然，还有取并集、差集等操作

使用`SPOP`，可以从集合中弹出一个元素，很方便模拟特定问题。比如，要实现一个基于web的扑克游戏，使用首字母来表示花色，`梅花`(C)lubs, `方块`(D)iamonds, `红桃`(H)earts, `黑桃`(S)pades:

```redis
>  sadd deck C1 C2 C3 C4 C5 C6 C7 C8 C9 C10 CJ CQ CK
   D1 D2 D3 D4 D5 D6 D7 D8 D9 D10 DJ DQ DK H1 H2 H3
   H4 H5 H6 H7 H8 H9 H10 HJ HQ HK S1 S2 S3 S4 S5 S6
   S7 S8 S9 S10 SJ SQ SK
   (integer) 52
```

<!--
Now we want to provide each player with 5 cards. The SPOP command removes a random element, returning it to the client, so it is the perfect operation in this case.

However if we call it against our deck directly, in the next play of the game we'll need to populate the deck of cards again, which may not be ideal. So to start, we can make a copy of the set stored in the deck key into the game:1:deck key.

This is accomplished using SUNIONSTORE, which normally performs the union between multiple sets, and stores the result into another set. However, since the union of a single set is itself, I can copy my deck with:

Now I'm ready to provide the first player with five cards:

One pair of jacks, not great...
-->

现在为每个玩家提供5张牌，使用`SPOP`可以随机弹出一个元素并返回给客户端，在这种情况下，`Set`就比较适用。

然而，当要为另一玩家发牌时，即需要再一次调用`SPOP`，那么需要重新生成整个牌面，这不是一个好的思路。其解决方法是将整个牌面复制一份保存到`game:1:deck`这个key。

使用`SUNIONSTORE`就可以完成上述操作。对于多个不同集合，可以执行并集操作，并将其结果保存到另一个集合。又因为单个集合的并集操作后就是他本身，那我们可以这么做：

```redis
> sunionstore game:1:deck deck
(integer) 52
```

现在我们为第一个玩家提供5张牌：

```redis
> spop game:1:deck
"C6"
> spop game:1:deck
"CQ"
> spop game:1:deck
"D1"
> spop game:1:deck
"CJ"
> spop game:1:deck
"SJ"
```

只有一对`J`，看来不是什么好牌...

<!--
This is a good time to introduce the set command that provides the number of elements inside a set. This is often called the cardinality of a set in the context of set theory, so the Redis command is called SCARD.
-->

现在介绍另一个命令，可用于返回集合中元素的剩余数。在集合理论中有基数的概念，所以在`Redis`中，命令叫做`SCARD`。

```redis
> scard game:1:deck
(integer) 47
```

<!--
The math works: 52 - 5 = 47.

When you need to just get random elements without removing them from the set, there is the SRANDMEMBER command suitable for the task. It also features the ability to return both repeating and non-repeating elements.
-->

运算方式： 52 - 5 = 47

当你需要从一个集合中取一个元素，并且需要删除元素，可以使用`SRANDMEMBER`。他还具有返回重复和非重复的元素。

<!-- Redis Sorted sets -->

## Redis 有序集合

<!--
Sorted sets are a data type which is similar to a mix between a Set and a Hash. Like sets, sorted sets are composed of unique, non-repeating string elements, so in some sense a sorted set is a set as well.

However while elements inside sets are not ordered, every element in a sorted set is associated with a floating point value, called the score (this is why the type is also similar to a hash, since every element is mapped to a value).

Moreover, elements in a sorted sets are taken in order (so they are not ordered on request, order is a peculiarity of the data structure used to represent sorted sets). They are ordered according to the following rule:
-->

有序集合就像是集合与哈希的混合体。有序集合是由不重复，唯一的字符串元素组成，这和集合相似。所以在某种意义下，有序集合就是一个集合。

在集合中的元素是无序，然而在有序集合中的每一个元素根据一个浮点数排序，这个浮点数叫`权值`(这也是为什么像哈希，因为每个元素都对应一个值)。

再说一点，有序集合中的元素是被动按序排列(因为他们不是按插入的顺序进行排序)。进行排序的规则如下：

<!--
If A and B are two elements with a different score, then A > B if A.score is > B.score.
If A and B have exactly the same score, then A > B if the A string is lexicographically greater than the B string. A and B strings can't be equal since sorted sets only have unique elements.

Let's start with a simple example, adding a few selected hackers names as sorted set elements, with their year of birth as "score".
-->

* 如果`A`和`B`有两个不同的权值，如果`A.socre > B.socre`，则`A > B`
* 如果`A`和`B`具有相同的权值，则按`A`与`B`的字符顺序进行排序。因为有序集合只能保存唯一的元素，`A`与`B`不能相等。

先看下这个简单的示例，根据出生年，为一个`hackers`集合，进行排序。

```redis
> zadd hackers 1940 "Alan Kay"
(integer) 1
> zadd hackers 1957 "Sophie Wilson"
(integer) 1
> zadd hackers 1953 "Richard Stallman"
(integer) 1
> zadd hackers 1949 "Anita Borg"
(integer) 1
> zadd hackers 1965 "Yukihiro Matsumoto"
(integer) 1
> zadd hackers 1914 "Hedy Lamarr"
(integer) 1
> zadd hackers 1916 "Claude Shannon"
(integer) 1
> zadd hackers 1969 "Linus Torvalds"
(integer) 1
> zadd hackers 1912 "Alan Turing"
(integer) 1
```

<!--
As you can see ZADD is similar to SADD, but takes one additional argument (placed before the element to be added) which is the score. ZADD is also variadic, so you are free to specify multiple score-value pairs, even if this is not used in the example above.

With sorted sets it is trivial to return a list of hackers sorted by their birth year because actually they are already sorted.

Implementation note: Sorted sets are implemented via a dual-ported data structure containing both a skip list and a hash table, so every time we add an element Redis performs an O(log(N)) operation. That's good, but when we ask for sorted elements Redis does not have to do any work at all, it's already all sorted:
-->

你也看到`ZADD`与`SADD`相似，但多一个可选的参数(该参数位于要加的元素前)，该参数就是权值。`ZADD`命令是可变长传参的，所以你可以指定多个`score-value`对。尽管上述例子中没有使用到，但你确实可以这么做。

使用有序集合可以返回一个已经排好序的`hackers`集合。

实现：有序集合是通过双端输入输出的数据结构，他包含一个列表和一个哈希表，所以每次加一个元素的时间复杂度为`O(log(N))`。但是在取元素时，因为已经排好序了，就没有其他需要做的工作了。

```redis
> zrange hackers 0 -1
1) "Alan Turing"
2) "Hedy Lamarr"
3) "Claude Shannon"
4) "Alan Kay"
5) "Anita Borg"
6) "Richard Stallman"
7) "Sophie Wilson"
8) "Yukihiro Matsumoto"
9) "Linus Torvalds"
```

<!--
Note: 0 and -1 means from element index 0 to the last element (-1 works here just as it does in the case of the LRANGE command).

What if I want to order them the opposite way, youngest to oldest? Use ZREVRANGE instead of ZRANGE:
-->

注意：0和-1分别表示第一个和最后一个元素的下标(与`LRANGE`命令中表示的模式一样)

如果你想进行返回排序，那就使用`ZREVRANGE`:

```redis
> zrevrange hackers 0 -1
1) "Linus Torvalds"
2) "Yukihiro Matsumoto"
3) "Sophie Wilson"
4) "Richard Stallman"
5) "Anita Borg"
6) "Alan Kay"
7) "Claude Shannon"
8) "Hedy Lamarr"
9) "Alan Turing"
```

<!--
It is possible to return scores as well, using the WITHSCORES argument:
-->

还可以使用`WITHSCORES`参数，将权值返回

```redis
> zrange hackers 0 -1 withscores
1) "Alan Turing"
2) "1912"
3) "Hedy Lamarr"
4) "1914"
5) "Claude Shannon"
6) "1916"
7) "Alan Kay"
8) "1940"
9) "Anita Borg"
10) "1949"
11) "Richard Stallman"
12) "1953"
13) "Sophie Wilson"
14) "1957"
15) "Yukihiro Matsumoto"
16) "1965"
17) "Linus Torvalds"
18) "1969"
```

<!--
Operating on ranges
-->

## 区间操作

<!--
Sorted sets are more powerful than this. They can operate on ranges. Let's get all the individuals that were born up to 1950 inclusive. We use the ZRANGEBYSCORE command to do it:
-->

有序集合强大的地方不止于此。我们可以操作有序集合的区间。现在我们通过命令可得到出生年小于等于1950的所有个体。我们使用`ZRANGEBYSCORE`来实现：


```redis
> zrangebyscore hackers -inf 1950
1) "Alan Turing"
2) "Hedy Lamarr"
3) "Claude Shannon"
4) "Alan Kay"
5) "Anita Borg"
```

<!--
We asked Redis to return all the elements with a score between negative infinity and 1950 (both extremes are included).

It's also possible to remove ranges of elements. Let's remove all the hackers born between 1940 and 1960 from the sorted set:
-->

我们要求Redis返回权值在负无穷到1950的元素(包含两个端值)


还可以通过区间来移除元素，让我们将1940到1960权值的元素移除：

```redis
> zremrangebyscore hackers 1940 1960
(integer) 4
```

<!--
ZREMRANGEBYSCORE is perhaps not the best command name, but it can be very useful, and returns the number of removed elements.

Another extremely useful operation defined for sorted set elements is the get-rank operation. It is possible to ask what is the position of an element in the set of the ordered elements.
-->

`ZREMRANGEBYSCORE`也并不是最好的命令，但他十分有用，其结果为删除元素的个数。

`get-rank`(取权值)操作是另一个非常好用的操作。他可以返回给定元素在有序集合中的位置

```redis
> zrank hackers "Anita Borg"
(integer) 4
```

<!--
The ZREVRANK command is also available in order to get the rank, considering the elements sorted a descending way.
-->

`ZREVRANK`命令同样可以取得元素位置，但其元素的顺序是降序的

<!--
Lexicographical scores
With recent versions of Redis 2.8, a new feature was introduced that allows getting ranges lexicographically, assuming elements in a sorted set are all inserted with the same identical score (elements are compared with the C memcmp function, so it is guaranteed that there is no collation, and every Redis instance will reply with the same output).

The main commands to operate with lexicographical ranges are ZRANGEBYLEX, ZREVRANGEBYLEX, ZREMRANGEBYLEX and ZLEXCOUNT.

For example, let's add again our list of famous hackers, but this time use a score of zero for all the elements:
-->


## Lexicographical scores

当前`Redis`版本为`2.8`，加入了一个新特性。假设所有元素都使用相同的权值(使用`C`语言中的`memcmp`方法，返回值一样的元素)，就生成一组按字典顺序的区间。

操作字典顺序区间的命令有`ZRANGEBYLEX`、`ZREVRANGEBYLEX`、`ZREMRANGEBYLEX`、`ZLEXCOUNT`

如下，这次我们再向`hacker`集合增加几名黑客，但各元素的权值均为0


```redis
> zadd hackers 0 "Alan Kay" 0 "Sophie Wilson" 0 "Richard Stallman" 0
  "Anita Borg" 0 "Yukihiro Matsumoto" 0 "Hedy Lamarr" 0 "Claude Shannon"
  0 "Linus Torvalds" 0 "Alan Turing"
```

<!--
Because of the sorted sets ordering rules, they are already sorted lexicographically:
-->

根据有序集合排序的规则，他们已经按字典顺序进行排序

```redis
> zrange hackers 0 -1
1) "Alan Kay"
2) "Alan Turing"
3) "Anita Borg"
4) "Claude Shannon"
5) "Hedy Lamarr"
6) "Linus Torvalds"
7) "Richard Stallman"
8) "Sophie Wilson"
9) "Yukihiro Matsumoto"
```

<!--
Using ZRANGEBYLEX we can ask for lexicographical ranges:
-->

使用`ZRANGEBYLEX`命令，可以用来操作字典顺序区间

```redis
> zrangebylex hackers [B [P
1) "Claude Shannon"
2) "Hedy Lamarr"
3) "Linus Torvalds"
```

<!--
Ranges can be inclusive or exclusive (depending on the first character), also string infinite and minus infinite are specified respectively with the + and - strings. See the documentation for more information.
-->

<!--
This feature is important because it allows us to use sorted sets as a generic index. For example, if you want to index elements by a 128-bit unsigned integer argument, all you need to do is to add elements into a sorted set with the same score (for example 0) but with an 16 byte prefix consisting of the 128 bit number in big endian. Since numbers in big endian, when ordered lexicographically (in raw bytes order) are actually ordered numerically as well, you can ask for ranges in the 128 bit space, and get the element's value discarding the prefix.
-->

这个特性相当重要，因为我们可以像使用索引的方法操作有序集合。比如，如果你要保存一组128位的无符号整型元素，你可以将这些元素加入到有序集合，并且这些元素的权值都是一样的，但因为其16个字节可以理解成是已经按字节顺序进行存储了，那么加入的元素也已经排好序了。

<!--
If you want to see the feature in the context of a more serious demo, check the Redis autocomplete demo.
-->

如果你想查看更多的示例，请浏览[autocomplete demo](http://autocomplete.redis.io/)


<!--
Just a final note about sorted sets before switching to the next topic. Sorted sets' scores can be updated at any time. Just calling ZADD against an element already included in the sorted set will update its score (and position) with O(log(N)) time complexity. As such, sorted sets are suitable when there are tons of updates.
-->

有序集合的权值可以在任何时刻改变。只需要再一次调用`ZADD`，增加一个已有元素，则该元素的权值就可以被更新，其时间复杂度为`O(log(N))`。这样有序集合也非常适用于更新频繁的场景。


<!---
Bitmaps
Bitmaps are not an actual data type, but a set of bit-oriented operations defined on the String type. Since strings are binary safe blobs and their maximum length is 512 MB, they are suitable to set up to 232 different bits.

Bit operations are divided into two groups: constant-time single bit operations, like setting a bit to 1 or 0, or getting its value, and operations on groups of bits, for example counting the number of set bits in a given range of bits (e.g., population counting).

One of the biggest advantages of bitmaps is that they often provide extreme space savings when storing information. For example in a system where different users are represented by incremental user IDs, it is possible to remember a single bit information (for example, knowing whether a user wants to receive a newsletter) of 4 billion of users using just 512 MB of memory.

Bits are set and retrieved using the SETBIT and GETBIT commands:
-->

## Bitmaps

`Bitmaps`并不是真正意义上的数据类型，而是一组定义在字符串类型上针对`位`的一组操作。如将位设置为0、设置为1、取值、多位操作，比如计算给定区间内`位`的值。

`Bitmaps`最大好处之一，就是可以提供一个极小空间来保存信息。比如在一个系统中，不同的用户都有一个不同的ID，那么系统有需要去记住`只占一位`的信息(比如用记是否想要接收新闻，这样40亿的用户只需要512M内存)

```redis
> setbit key 10 1
(integer) 1
> getbit key 10
(integer) 1
> getbit key 11
(integer) 0
```

<!--
The SETBIT command takes as its first argument the bit number, and as its second argument the value to set the bit to, which is 1 or 0. The command automatically enlarges the string if the addressed bit is outside the current string length.

GETBIT just returns the value of the bit at the specified index. Out of range bits (addressing a bit that is outside the length of the string stored into the target key) are always considered to be zero.

There are three commands operating on group of bits:
-->

`SETBIT`使用第一个参数作为其位上值，第二个参数用于设置是该位是1还是0

`GETBIT`返回指定索引上的的`位值`。出了区间()，可以理解为0。

按位有三种命令操作：

<!--
1. BITOP performs bit-wise operations between different strings. The provided operations are AND, OR, XOR and NOT.
2. BITCOUNT performs population counting, reporting the number of bits set to 1.
3. BITPOS finds the first bit having the specified value of 0 or 1.
-->


* `BITOP`：提供按位操作，如`AND`、`OR`、`XOR`和`NOT`
* `BITCOUNT`：计算某区间内1的个数
* `BITPOS`：指定0或1，返回第一次相等的位置

<!--
Both BITPOS and BITCOUNT are able to operate with byte ranges of the string, instead of running for the whole length of the string. The following is a trivial example of BITCOUNT call:
-->

`BITPOS`和`BITCOUNT`也同样适用于字符串操作，这样就可以代替字符串求长度的操作。

```redis
> setbit key 0 1
(integer) 0
> setbit key 100 1
(integer) 0
> bitcount key
(integer) 2
```

<!--
Common use cases for bitmaps are:
-->

bitmaps的使用场景：

<!--
1. Real time analytics of all kinds.
2. Storing space efficient but high performance boolean information associated with object IDs.
-->

* 实时分析
* 对于只存ID和一位信息的场景，提供了高性能的存储空间

<!--
For example imagine you want to know the longest streak of daily visits of your web site users. You start counting days starting from zero, that is the day you made your web site public, and set a bit with SETBIT every time the user visits the web site. As a bit index you simply take the current unix time, subtract the initial offset, and divide by 3600*24.

This way for each user you have a small string containing the visit information for each day. With BITCOUNT it is possible to easily get the number of days a given user visited the web site, while with a few BITPOS calls, or simply fetching and analyzing the bitmap client-side, it is possible to easily compute the longest streak.

Bitmaps are trivial to split into multiple keys, for example for the sake of sharding the data set and because in general it is better to avoid working with huge keys. To split a bitmap across different keys instead of setting all the bits into a key, a trivial strategy is just to store M bits per key and obtain the key name with bit-number/M and the Nth bit to address inside the key with bit-number MOD M.
-->

如果你想知道浏览你的网站用户的时长列表。你的网站发布到公网后，将开始时间设置为0，当用户访问你的网站后，使用`SETBIT`设置一位信息，信息存储的是当前时间减去初始时间，再除以`3600*24`。

<!--
HyperLogLogs

A HyperLogLog is a probabilistic data structure used in order to count unique things (technically this is referred to estimating the cardinality of a set). Usually counting unique items requires using an amount of memory proportional to the number of items you want to count, because you need to remember the elements you have already seen in the past in order to avoid counting them multiple times. However there is a set of algorithms that trade memory for precision: you end with an estimated measure with a standard error, which in the case of the Redis implementation is less than 1%. The magic of this algorithm is that you no longer need to use an amount of memory proportional to the number of items counted, and instead can use a constant amount of memory! 12k bytes in the worst case, or a lot less if your HyperLogLog (We'll just call them HLL from now) has seen very few elements.

HLLs in Redis, while technically a different data structure, are encoded as a Redis string, so you can call GET to serialize a HLL, and SET to deserialize it back to the server.

Conceptually the HLL API is like using Sets to do the same task. You would SADD every observed element into a set, and would use SCARD to check the number of elements inside the set, which are unique since SADD will not re-add an existing element.

While you don't really add items into an HLL, because the data structure only contains a state that does not include actual elements, the API is the same:
-->

<!--
1. Every time you see a new element, you add it to the count with PFADD.
2. Every time you want to retrieve the current approximation of the unique elements added with PFADD so far, you use the PFCOUNT.
-->

```redis
> pfadd hll a b c d
(integer) 1
> pfcount hll
(integer) 4
```

<!--
An example of use case for this data structure is counting unique queries performed by users in a search form every day.

Redis is also able to perform the union of HLLs, please check the full documentation for more information.
-->



<!--
Other notable features
There are other important things in the Redis API that can't be explored in the context of this document, but are worth your attention:
1. It is possible to iterate the key space of a large collection incrementally.
2. It is possible to run Lua scripts server side to improve latency and bandwidth.
Redis is also a Pub-Sub server.
-->

## 其他有用特性

还有一些重要的内容没有表述在这篇文档中，但也值得你引起你的注意：

* 在大的集合中，key可以重复使用
* 可以在服务端跑Lua脚本，`Redis`也可以做为`Pub-Sub`服务器