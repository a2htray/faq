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

## INCR

> 值自增加1

```redis
> SET integer:11 100
OK
> INCR integer:11
(integer) 101
> GET integer:11
"101"
```

## GETSET

> 设置一个新的值，并将旧结果返回

```redis
> SET time:1 old
OK
> GETSET time:1 new
"old"
> GET time:1
"new"
```

## MSET

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

## MGET

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

## DEL

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

## LPUSH

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

### RPUSH

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

## RPOP

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

## LTRIM

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