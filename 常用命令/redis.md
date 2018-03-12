# Redis命令

## SET

> 设置key-value，也可直接替换已有的value

```redis
> SET mykey myval
OK
```

> 或该`mykey`存在，则不进行设置

```redis
> SET mykey myval
OK
> SET mykey newval nx
(nil)
```

> 或该`mykey`存在，才进行设置

```redis
> SET mykey myval
OK
> SET mykey newval xx
OK
```

> 当`mykey`不存在时，使用`nx`选项，也可以进行设置

```redis
> GET mykey
(nil)
> SET mykey myval nx
OK
```

> 当`mykey`不存在时，使用`xx`选项，不进行设置

```redis
> GET mykey
(nil)
> SET mykey myval xx
(nil)
```

> 设置一个过期时间，使用`ex`选项

```redis
> SET expire:1 5 ex 10
OK
> GET expire:1
"5"
> GET expire:1 // 10秒后查看
(nil)
```

## INCR - Increase

> 值自增加1

```redis
> SET integer:11 100
OK
> INCR integer:11
(integer) 101
> GET integer:11
"101"
```

## GETSET - Get And Set

> 设置一个新的值，并将旧结果返回

```redis
> SET time:1 old
OK
> GETSET time:1 new
"old"
> GET time:1
"new"
```

## MSET - Multiple Set

> 设置多个key

```redis
> MGET key1 key2 key3
1) (nil)
2) (nil)
3) (nil)
> MSET key1 1 key2 2 key3 3
OK
> MGET key1 key2 key3
1) "1"
2) "2"
3) "3"
```

## MGET - Multiple Get

> 读取多个key

```redis
> MGET key1 key2 key3
1) (nil)
2) (nil)
3) (nil)
> MSET key1 1 key2 2 key3 3
OK
> MGET key1 key2 key3
1) "1"
2) "2"
3) "3"
```

## EXISTS

> 判断是否存在指定key

```redis
> SET key1 value1
OK
> EXISTS key1 // key1存在，返回1
(integer) 1
> EXISTS key2 // key2不存在，返回0
(integer) 0
```

## DEL - Delete

> 删除指定key

```redis
> EXISTS key1
(integer) 1
> DEL key1
(integer) 1
> EXISTS key5 // key5不存在
(integer) 0
> DEL key5 
(integer) 0 // 执行删除操作返回0
```

## TYPE

> 查看指定key保存值的类型

```redis
> SET spec:1 collapse
OK
> TYPE spec:1
"string"
> EXISTS spec:2
(integer) 0
> TYPE spec:2
"none"
```

## EXPIRE

> 设置一个过期时间

```redis
> SET expire:1 "i will disappear"
OK
> EXPIRE expire:1 5 // 设置5秒后过期
(integer) 1
> GET expire:1 // 5秒后执行
(nil)
```

## LPUSH - List Push

> 首部新增一个元素，返回插入后列表的长度

```redis
> LPUSH list:2 a
(integer) 1
> LPUSH list:2 a
(integer) 2
> LPUSH list:2 b
(integer) 3
> LPUSH list:2 b
(integer) 4
> LRANGE list:2 0 3
1) "b"
2) "b"
3) "a"
4) "a"
```

> 插入多个元素，返回插入后列表的长度

```redis
> LPUSH list:9 1 2 3 4 5 6 7 8 // 先执行插入1的操作，最后执行插入8的操作
(integer) 8
> LRANGE list:9 0 -1
1) "8"
2) "7"
3) "6"
4) "5"
5) "4"
6) "3"
7) "2"
8) "1"
```

### RPUSH - Right Push

> 尾部新增一个元素，返回插入后列表的长度

```redis
> RPUSH list:3 a
(integer) 1
> RPUSH list:3 a
(integer) 2
> RPUSH list:3 B
(integer) 3
> RPUSH list:3 B
(integer) 4
> LRANGE list:3 0 4
1) "a"
2) "a"
3) "B"
4) "B"
```

## RPOP - Right Pop

> 移除一个元素，并返回该元素

```redis
> RPUSH list:10 a b c d e f
(integer) 6
> RPOP list:10
"f"
> LRANGE list:10 0 4
1) "a"
2) "b"
3) "c"
4) "d"
5) "e"
```

## LTRIM - List Trim

> 截取列表，设置指定区间内的元素为列表的新值

```redis
> LPUSH list:19 1 2 3 4 5 6 7
(integer) 7
> LTRIM list:19 0 4
OK
> LRANGE list:19 0 4
1) "7"
2) "6"
3) "5"
4) "4"
5) "3"
```

## HMSET - Hash Multiple Set

> 创建哈希

```redis
> HMSET grade:1 number 1001 total 54 floor 4
OK
```

## HGET - Hash Get
 
> 读取特定信息

```redis
> HMSET grade:1 number 1001 total 54 floor 4
OK
> HGET grade:1 number
"1001"
```

## HGETALL - Hash Get All

> 取哈希所有信息

```redis
> HMSET grade:1 number 1001 total 54 floor 4
OK
> HGETALL grade:1
1) "number"
2) "1001"
3) "total"
4) "54"
5) "floor"
6) "4"
```

## HINCRBY - Hash Increase By

> 特定字段自增，需要两个参数

```redis
> HMSET grade:1 number 1001 total 54 floor 4
OK
> HINCRBY grade:1 number 10
(integer) 1011
```

## SADD - Set Add

> 集合增加元素

```redis
> SADD myset 1 2 3
(integer) 3
> SMEMBERS myset
1. 3
2. 1
3. 2
```

## SMEMBERS - Set Members

> 取集合元素

```redis
> SADD set:1 a 1 b 2 c 3
(integer) 6
> SMEMBERS set:1
1) "b"
2) "1"
3) "a"
4) "c"
5) "3"
6) "2"
```

## SISMEMBER - Set Is Member

> 判断集合中是否存在特定元素

```redis
> SADD set:1 a 1 b 2 c 3
(integer) 6
> SMEMBERS set:1
1) "b"
2) "1"
3) "a"
4) "c"
5) "3"
6) "2"
> SISMEMBER set:1 a
(integer) 1
```

## SINTER - Set Intersection

> 集合取交集操作

```redis
> SADD set:11 a b c
(integer) 3
> SADD set:12 b c d
(integer) 3
> SINTER set:11 set:12
1) "b"
2) "c"
```

## SPOP - Set Pop

> 随机从集合中弹出一个元素，集合中，该元素被删除

```redis
> SADD set:13 a b a b
(integer) 2
> SPOP set:13
"b"
> SCARD set:13
1
```

## SRANDMEMBER - Set Random Member

> 随机从集合中取一个元素，原集合不变

```redis
> SADD set:13 a b a b
(integer) 2
> SRANDMEMBER set:13
"a"
> SCARD set:13
2
```