
### 数据库

> **选择合适的数据类型**

> **使用`CHAR(1)`代替`VARCHAR(1)`**

使用`VARCHAR(1)`存储 1 个长度的字符时，会使用额外的字节存储其他信息

> **当字符数固定时，请使用`CHAR`**

> **避免使用区域性的时间格式存储时间信息**

> **选择合适的字段作为索引，对查询条件字段加入索引**

如果表中所有字段都可能进行条件进行查询，那就把所有字段加上索引

对于进行`JOIN`查询时，请保证`JOIN`的两个字段都是索引，并且具有相同的数据类型

> **不要在作为索引的字段上应用函数**

这样就失去将其作为索引的意义

> **不要直接使用`SELECT *`，按需进行选择**

> **选择合适的数据库引擎**

读大于写：MyISAM

写大于读：InnoDB

> **尽可能设置为字段`NOT NULL`**

没有绝对需求，为字段设置`NOT NULL`

> **将 ip 地址设置为无符号的整型 UNSIGNED INT**

> **使用最小的规格、合适的数据类型**

## 代码

> **优化查询语句，以便利用到查询缓存**

```php
// 没有使用到查询缓存
$r = mysql_query("SELECT username FROM user WHERE signup_date >= CURDATE()");
 
// 使用到了查询缓存
$today = date("Y-m-d");
$r = mysql_query("SELECT username FROM user WHERE signup_date >= '$today'");
```

> **在只取一行时，加上`LIMIT 1`**

```php
// 良
$r = mysql_query("SELECT * FROM user WHERE state = 'Alabama'");
if (mysql_num_rows($r) > 0) {
    // ...
}

// 优
$r = mysql_query("SELECT 1 FROM user WHERE state = 'Alabama' LIMIT 1");
if (mysql_num_rows($r) > 0) {
    // ...
}
```

> **不要使用`ORDER BY RAND()`**

应选择在程序中进行随机的操作

> **使用预处理语句**

## 部署

> 公网上不可进行直接访问

## 参考

* [http://bigdata-madesimple.com/top-10-best-practices-in-mysql/](http://bigdata-madesimple.com/top-10-best-practices-in-mysql/)
* [https://code.tutsplus.com/tutorials/top-20-mysql-best-practices--net-7855](https://code.tutsplus.com/tutorials/top-20-mysql-best-practices--net-7855)
* [https://www.databasejournal.com/features/mysql/article.php/3918631/Top-10-MySQL-Best-Practices.htm](https://www.databasejournal.com/features/mysql/article.php/3918631/Top-10-MySQL-Best-Practices.htm)