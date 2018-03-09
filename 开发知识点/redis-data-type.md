> 原文[https://redis.io/topics/data-types-intro](https://redis.io/topics/data-types-intro)

# _An introduction to Redis data types and abstractions_

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

## _Redis keys_

_Redis keys are binary safe, this means that you can use any binary sequence as a key, from a string like "foo" to the content of a JPEG file. The empty string is also a valid key._

Redis的key是二进制安全的，这就意味着你可以拿任何二进制序列作为key，比如拿字符串`foo`可以和一个`JPEG`文件的内容关联起来。空的字符串同样是一个有效的key。

_A few other rules about keys:_

其他一些关于key的规则：

* _Very long keys are not a good idea. For instance a key of 1024 bytes is a bad idea not only memory-wise, but also because the lookup of the key in the dataset may require several costly key-comparisons. Even when the task at hand is to match the existence of a large value, hashing it (for example with SHA1) is a better idea, especially from the perspective of memory and bandwidth._
* _Very short keys are often not a good idea. There is little point in writing "u1000flw" as a key if you can instead write "user:1000:followers". The latter is more readable and the added space is minor compared to the space used by the key object itself and the value object. While short keys will obviously consume a bit less memory, your job is to find the right balance._
* _Try to stick with a schema. For instance "object-type:id" is a good idea, as in "user:1000". Dots or dashes are often used for multi-word fields, as in "comment:1234:reply.to" or "comment:1234:reply-to"._
* _The maximum allowed key size is 512 MB._

* 使用较长的key不是一个好主意。比如，一个长度为1024字节的是一个不好的主意，不仅仅是会存在内存溢出的情况，还因为在查找key的过程中进行多次字符的比较，这是不小的代价。当手头上的工作是匹配一个较长的值时，特别是从内存和值位宽的角度来说，将该值进行哈希(比如使用`SHA1`)是一个不错的主意。
* 非常短的key也不是一个好的主意。其中有一点值得说的是，如果你可以使用`user:1000:followers`代替`u1000flw`作为key。代替之后的key更具有可读性，而且与key对象本身和value对象所占空间相比，增加的空间就显得比较不重要了。但是短点的key很显然占用较少的内存，你的任务就是要找到一个两者的平衡点。
* 尽量将对象类型在key中进行体现。比如`object-type:id`就是一个不错的主意，像`user:1000`这样。点`.`和连接符`-`常常用于有个单词组成的字段，比如`comment:1234:reply.to`或`comment:1234:reply-to`
* 最大允许的key的大小是`512MB`

## _Redis Strings_

## Redis 字符串

_The Redis String type is the simplest type of value you can associate with a Redis key. It is the only data type in Memcached, so it is also very natural for newcomers to use it in Redis._

你能想到可以作为Redis的key值中，字符串类型是最简单的。这也是唯一在内存缓存的数据类型，所以对在Redis上使用该类型的新手来说也是非常自然的。

_Since Redis keys are strings, when we use the string type as a value too, we are mapping a string to another string. The string data type is useful for a number of use cases, like caching HTML fragments or pages._

因为Redis的key是字符串类型，所以我们也用字符串来表示value，我们可以将一个字符串映射到另一个字符串。字符串作为value，在许多使用案例中都非常有用，如可以用于缓存HTML片段或页面。

_Let's play a bit with the string type, using redis-cli (all the examples will be performed via redis-cli in this tutorial)._

通过使用`redis-cli`(教程中的所有示例通过`redis-cli`被执行)，让我们来尝试一下字符串类型。

```redis
> set mykey somevalue
OK
> get mykey
"somevalue"
```

_As you can see using the SET and the GET commands are the way we set and retrieve a string value. Note that SET will replace any existing value already stored into the key, in the case that the key already exists, even if the key is associated with a non-string value. So SET performs an assignment._

如你所见，使用`SET`和`GET`命令是我们设置和获取一个字符串值的方法。值得注意的是，`SET`会替换任何已经存在，且保存于该key中的值，在这种情况下，都是表示已经存在了的key，即使这个key映射到的是一个非字符串的值。所以`SET`命令执行的是一个分配的任务。

_Values can be strings (including binary data) of every kind, for instance you can store a jpeg image inside a value. A value can't be bigger than 512 MB._

value可以是表示各个类型的字符串(包括二进制数据)，比如你可以在value中保存一张`jpeg`的图片。value的大小不能超过512MB。

_The SET command has interesting options, that are provided as additional arguments. For example, I may ask SET to fail if the key already exists, or the opposite, that it only succeed if the key already exists:_

`SET`命令也有一些有趣的选项，并以歌德参数的形式给出。比如，你可以希望当key值存在时不进行设置，或者反过来，当key存在时才进行设置。

```redis
> set mykey newval nx
(nil)
> set mykey newval xx
OK
```

_Even if strings are the basic values of Redis, there are interesting operations you can perform with them. For instance, one is atomic increment:_


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

_The `INCR command parses the string value as an integer, increments it by one, and finally sets the obtained value as the new value. There are other similar commands like INCRBY, DECR and DECRBY. Internally it's always the same command, acting in a slightly different way._

`INCR`命令将字符串值解析成一个整型，并自增加1，最终将得到的值设置为新的值。还有其他类似的命令，如`INCRBY`，`DECR`，`DECRBY`。在内部实现看来其实是相同的命令，但在执行过程中有微微的不同。

_What does it mean that INCR is atomic? That even multiple clients issuing INCR against the same key will never enter into a race condition. For instance, it will never happen that client 1 reads "10", client 2 reads "10" at the same time, both increment to 11, and set the new value to 11. The final value will always be 12 and the read-increment-set operation is performed while all the other clients are not executing a command at the same time._

为什么说`INCR`操作是原子性的？那就是即使有多个客户端对同一个key进行自增操作，并不会造成拥堵的状态。比如，当两个客户端在同一时刻将同一个值自增到`11`，那么两个客户端同时读到`10`是不会发生的。最终的值总会是`12`，只有在相同的时刻，在没有其他执行命令的情况下，才会执行`读取-自增-设置`操作。

_There are a number of commands for operating on strings. For example the GETSET command sets a key to a new value, returning the old value as the result. You can use this command, for example, if you have a system that increments a Redis key using INCR every time your web site receives a new visitor. You may want to collect this information once every hour, without losing a single increment. You can GETSET the key, assigning it the new value of "0" and reading the old value back._

有很多关于字符串的命令。比如`GETSET`，设置一个新的值，并将旧的值作为结果返回。如果你的系统在每一时刻需要使用`INCR`用于表示又有一个新的访问者。这类信息你可以要每小时收集一次，并且不要丢掉一个访问者数。你可以使用`GETSET`，将`0`作为最新的值，并将读取旧的值。

_The ability to set or retrieve the value of multiple keys in a single command is also useful for reduced latency. For this reason there are the MSET and MGET commands:_

为了减少延迟，可以在单一个命令中同时设置或读取多个key，这个能力相当有用。基于这个原因，就有了`MSET`和`MGET`：

```redis
> mset a 10 b 20 c 30
OK
> mget a b c
1) "10"
2) "20"
3) "30"
```

_When MGET is used, Redis returns an array of values._

当使用`MGET`时，会返回一个数组。

## Altering and querying the key space

## key的修改和查询

_There are commands that are not defined on particular types, but are useful in order to interact with the space of keys, and thus, can be used with keys of any type._

还有一些不是定义在特定类型上的命令，在于key的交互中非常有用，因此，不管该key值映射到何种类型的值，都可以使用。

_For example the EXISTS command returns 1 or 0 to signal if a given key exists or not in the database, while the DEL command deletes a key and associated value, whatever the value is._

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

_From the examples you can also see how DEL itself returns 1 or 0 depending on whether the key was removed (it existed) or not (there was no such key with that name)._

从上述的例子中可以看出，执行`DEL`操作会根据当前key是否存在，如果存在则被删除返回1，如果不存在则返回0。

_There are many key space related commands, but the above two are the essential ones together with the TYPE command, which returns the kind of value stored at the specified key:_

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

## _Redis expires: keys with limited time to live_

## key的过期时间

_Before continuing with more complex data structures, we need to discuss another feature which works regardless of the value type, and is called Redis expires. Basically you can set a timeout for a key, which is a limited time to live. When the time to live elapses, the key is automatically destroyed, exactly as if the user called the DEL command with the key._

在继续处理更复杂的数据类型前，我们需要来讨论另一种特性，这个特性适用于任何类型，叫做`过期时间`。最基本的事，你可以为一个key设置一个过期时间，这个key只在限定的时间内有效。当超过这个限定时间内，就会销毁这个key，就好像用户对这个key执行了`DEL`操作一样。

_A few quick info about Redis expires:_

快速了解下关于`过期时间`的知识：

* They can be set both using seconds or milliseconds precision.
* However the expire time resolution is always 1 millisecond.
* Information about expires are replicated and persisted on disk, the time virtually passes when your Redis server remains stopped (this means that Redis saves the date at which a key will expire).

* 在精确度上，可以设置为秒级别或毫秒级别
* 然而过期时间总是`1毫秒`
* 过期时间的信息是可再生的保存在磁盘上，即使`Redis`服务器停止了，也是按真实的时间来计算(这就意味着Redis上保存了这个key将要过期的时间)

_Setting an expire is trivial:_

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

_The key vanished between the two GET calls, since the second call was delayed more than 5 seconds. In the example above we used EXPIRE in order to set the expire (it can also be used in order to set a different expire to a key already having one, like PERSIST can be used in order to remove the expire and make the key persistent forever). However we can also create keys with expires using other Redis commands. For example using SET options:_

因为两调用`GET`时间间隔大于5，所以`key`已经消失了。在上述的例子中使用`EXPIRE`来设置一个过期时间(这个方法也同样适用于为已经有过期时间的key设置一个新的过期时间)，就好像使用`PERSIST`来移除过期时间，让其永远都存在。然而，我们同样可以使用别的`Redis`命令来创建带有过期时间的key。比如使用`SET`命令选项：

```redis
> set key 100 ex 10
OK
> ttl key
(integer) 9
```

_The example above sets a key with the string value 100, having an expire of ten seconds. Later the TTL command is called in order to check the remaining time to live for the key._

这述的例子中，我们创建一个key，这个key保存了一个字符串值`100`，并且有`10`秒的过期时间。接下来，使用`TTL`命令来检查这个key还能保存多长时间。

_In order to set and check expires in milliseconds, check the PEXPIRE and the PTTL commands, and the full list of SET options._

为了以毫秒的形式来设置和检查某个key，可以使用`PEXPIRE`和`PTTL`。也可以使用`SET`命令中的`px`选项。

## _Redis Lists_

## Redis List

_To explain the List data type it's better to start with a little bit of theory, as the term List is often used in an improper way by information technology folks. For instance "Python Lists" are not what the name may suggest (Linked Lists), but rather Arrays (the same data type is called Array in Ruby actually)._

因为在不同的技术分支中，`List`经常以一种不恰当的方式使用，为了解释`List`这种数据类型，最好还是从一点理论知识开始讲起。比如，在`Python Lists`并不像他名字提示的这样(链式线性表)，他其实是一个数组(在`Ruby`中，与之相同的数据类型确实叫做数组)。

_From a very general point of view a List is just a sequence of ordered elements: 10,20,1,2,3 is a list. But the properties of a List implemented using an Array are very different from the properties of a List implemented using a Linked List._

从一个常规的视角来看，一个`List`就是一系列有序的序列：10,20,1,2,3，就是一个`List`。但是使用数组实现的`List`和使用链式线性表表实现的`List`有很大的不同之处。

_Redis lists are implemented via Linked Lists. This means that even if you have millions of elements inside a list, the operation of adding a new element in the head or in the tail of the list is performed in constant time. The speed of adding a new element with the LPUSH command to the head of a list with ten elements is the same as adding an element to the head of list with 10 million elements._

`Redis`中的`List`是通过链式线性表，这就意味着，即使在你的列表中有多如百万计的元素，在首尾新增一个新元素的操作也能马上执行。使用`LPUSH`命令对一个只有10个元素的`List`和向一个有1000万元素的列表进行操作，在他们的首部插入一个新的元素，他们的速度是一样的。

_What's the downside? Accessing an element by index is very fast in lists implemented with an Array (constant time indexed access) and not so fast in lists implemented by linked lists (where the operation requires an amount of work proportional to the index of the accessed element)._

那有什么不好的特性吗？使用数组实现的`List`，通过索引方式访问是非常快的，但在以链式线性表实现的`List`中就没有那么快(该操作需要大量的工作来指定元素，工作规模与`List`的长度成正比)

> 数组具有随机存取的优势，链式表首尾操作，时间复杂度为`O(1)`

_Redis Lists are implemented with linked lists because for a database system it is crucial to be able to add elements to a very long list in a very fast way. Another strong advantage, as you'll see in a moment, is that Redis Lists can be taken at constant length in constant time._

`Redis List`是使用链式线性表来实现的，因为对于一个数据库系统来说，其最为关键的能力是，在尽可能快的时间内，在非常长的列表中新增一个元素。一会儿你会看这种方式的其他优势，那就是`Redis List`可以在常数阶的时间内得到常数阶的长度。

_When fast access to the middle of a large collection of elements is important, there is a different data structure that can be used, called sorted sets. Sorted sets will be covered later in this tutorial._

当你的应用中，更看重如何在一个较大集合中更快地获取其中间的元素，也有一个不同的数据结构`有序List`可以使用。`有序List`会在后面的教程中提及。

## First steps with Redis Lists

## 走出第一步

_The LPUSH command adds a new element into a list, on the left (at the head), while the RPUSH command adds a new element into a list ,on the right (at the tail). Finally the LRANGE command extracts ranges of elements from lists:_

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

_Note that LRANGE takes two indexes, the first and the last element of the range to return. Both the indexes can be negative, telling Redis to start counting from the end: so -1 is the last element, -2 is the penultimate element of the list, and so forth._

注意，`LRANGE`命令使用两个下标，分别表示第一个元素和第二个元素在在列表中的下标。这个两个下标都是可以为负数，这样就告诉`Redis`从尾部开始计算：所以`-1`表示最后一个元素，`-2`表示倒数第二个元素，以此类推。

_As you can see RPUSH appended the elements on the right of the list, while the final LPUSH appended the element on the left._

正如你所看到的一样，`RPUSH`将元素新增于`list`的右边，同时，最后一个`LPUSH`将元素新增于`list`的左边。

_Both commands are variadic commands, meaning that you are free to push multiple elements into a list in a single call:_

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

_An important operation defined on Redis lists is the ability to pop elements. Popping elements is the operation of both retrieving the element from the list, and eliminating it from the list, at the same time. You can pop elements from left and right, similarly to how you can push elements in both sides of the list:_

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

_We added three elements and popped three elements, so at the end of this sequence of commands the list is empty and there are no more elements to pop. If we try to pop yet another element, this is the result we get:_

你向一个列表中新加3个元素，随后又移除了3个元素，所以执行这一系列命令后，这个列表是空的，列表中不存在任何元素。如果你有去执行`pop`操作，你会得到下面的结果：

```redis
> rpop mylist
(nil)
```

_Redis returned a NULL value to signal that there are no elements in the list._

Redis返回一个`NULL`值，用于标识在这个列表中已经没有元素了。

## _Common use cases for lists_

## Lists的常用用法

_Lists are useful for a number of tasks, two very representative use cases are the following:_

列表应用于很多的任务当中，非常有用。下面有两个非常有代表性的示例：

* _Remember the latest updates posted by users into a social network._
* _Communication between processes, using a consumer-producer pattern where the producer pushes items into a list, and a consumer (usually a worker) consumes those items and executed actions. Redis has special list commands to make this use case both more reliable and efficient._

* 在社交网站上，记得用户最近更新、发表的内容
* 进程之间进行通信，使用`消费者-生产者`的模式时，当一个生产者向列表中新增一些元素，另一方面有一个消费者(通常是woker)去处理这些元素，并执行相应的动作。`Redis`有特殊的列表命令，可以保证在这种使用场景下，可靠性更强，效率更高。

_For example both the popular Ruby libraries resque and sidekiq use Redis lists under the hood in order to implement background jobs._

如，有两个非常有名的`Ruby`库，`resque`和`sidekiq`，都在后台使用`Redis`的列表来实现后台任务。


_The popular Twitter social network takes the latest tweets posted by users into Redis lists._

著名的社交网站`Twitter`使用`Redis`列表，来实现提取最近更新的内容。

_To describe a common use case step by step, imagine your home page shows the latest photos published in a photo sharing social network and you want to speedup access._

为了一步一步描述常用用法，想像一下，你的社交主页展示了最近一次发表于照片圈的一张图片，然后你希望让访问速度变快。

* _Every time a user posts a new photo, we add its ID into a list with LPUSH._
* _When users visit the home page, we use LRANGE 0 9 in order to get the latest 10 posted items._

* 每一次用户发表一张新的图片，我们使用`LPUSH`将该图片的ID加入到列表中
* 当用户访问主页时，我们使用`LRANGE 0 9`，获得最近更新的10张图片

## Capped lists

## 伪固定长度列表

_In many use cases we just want to use lists to store the latest items, whatever they are: social network updates, logs, or anything else._

在多数的情况下，我们只是想用列表来保存最近更新的内容，不管他们是什么，社交网站的更新内容也好，日志或者其他什么东西也好，都可以。

_Redis allows us to use lists as a capped collection, only remembering the latest N items and discarding all the oldest items using the LTRIM command._

`Redis`允许你用列表创建一个伪固定长度的集合，只用来保存更新的`N`个元素，并通过`LTRIM`命令移除所以时间最久的元素。

_The LTRIM command is similar to LRANGE, but instead of displaying the specified range of elements it sets this range as the new list value. All the elements outside the given range are removed._

`LTRIM`命令与`LRANGE`有些相似，但与展示一个特定区间内的元素不同的是，他会将这个特定区间作为列表新的值。所有不在这个区间内的元素都会被移除。

_An example will make it more clear:_

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

_The above LTRIM command tells Redis to take just list elements from index 0 to 2, everything else will be discarded. This allows for a very simple but useful pattern: doing a List push operation + a List trim operation together in order to add a new element and discard elements exceeding a limit:_

上述的`LTRIM`告诉`Redis`只需要从列表中取下标为`0`到`2`的元素，其他的元素都会被丢弃。让一个简单但有用的模式得以行得通：一个新增`push`元素操作加一个截取`trim`元素操作，当超过一定限制后，可以实现每新加一个元素就可以移除一个元素：

```redis
LPUSH mylist <some element>
LTRIM mylist 0 999
```

_The above combination adds a new element and takes only the 1000 newest elements into the list. With LRANGE you can access the top items without any need to remember very old data._

上面的操作结合了新加一个元素和只取1000最新的元素。这样，你就可以使用`LRANGE`操作选取上面一些元素，而不需要去记住那些非常老的数据。

_Note: while LRANGE is technically an O(N) command, accessing small ranges towards the head or the tail of the list is a constant time operation._

注意：`LRANGE`是一个时间复杂度为`O(N)`的命令，从首部还尾部访问一个区间的元素是一个固定时间的操作。