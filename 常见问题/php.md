# 常见问题


## Composer 内存不足

> PHP Fatal error:  Allowed memory size of 1610612736 bytes exhausted (tried to allocate 268435456 bytes)

出现类似的，可通过修改`php.ini`中的`memory_limit`解决，也可以使用下面的命令：

```bash
php -d memory_limit=-1 composer.phar <...>
```

`memory_limit=-1`表示不限制。

在我机子上不行啊，然后将`mysqld`、`nginx`、`redis`服务关闭了，
