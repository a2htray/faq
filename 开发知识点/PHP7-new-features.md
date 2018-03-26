# PHP7 中的新特性

先给出一个问题，为什么`PHP7`那么特殊，从`5`到`7`又加入了哪些新特性？

现在就来总结下：

## 速度 Speed

在PHP中数组的的本质其实是一个有序的字典，字典具有键值对，而键值对的映射又是通过哈希表来实现，那么通过修改哈希表的实现从而提升了PHP中的性能。速度方面，PHP7是PHP5的两倍，而在内存消耗上，PHP7又少了大约2.5倍，性能得到很大的提升。

PHP使用C来实现，当中的每一个变量都是C中`zval结构`，且在PHP7中对`zval结构`也做了重构。

有愿意看`C`代码可以直接访问`参考`中第一、二、三链接，看下新的是如何实现的。(不才看不懂...)


## 类型声明

都知道，在编译型语言中，使用类型声明是很常见的是，应该需要在编译阶段进行代码检错，从而可为运行时的安全运行提供前提基础。

但现在，你可以在`PHP7`中使用类型声明，如下：

```php
<?php
function add(int $a, int $b)
{
    return $a + $b;
}

echo add(1, 2) . "\n"; // 3  正常执行
echo add(1.2, 8.8) . "\n"; // 9  正常执行，但发生的类型转换，1.2 -> 1  8.8 -> 8 

```

### 返回值类型声明

```php
function sub(float $a, float $b): int
{
    return $a - $b;
}

echo sub(8.9, 9) . "\n"; // 0
```

上例中，两个参数分别为`float`类型，返回值类型为`int`，则执行的顺序如下：

* 两个浮点数相减，得`-0.1`
* `-0.1`发生类型转换，变成了`int`类型的`0`，并返回

### 可选择强制约束

可以使用`declare(strict_types=1);`，强制使用类型声明中的变量类型，若

```php
<?php
declare(strict_types=1);

function add(int $a, int $b)
{
    return $a + $b;
}

echo add(1, 2) . "\n";
echo add(1.2, 8.8) . "\n";
```

产生如下的错误

```bash
Fatal error: Uncaught TypeError: Argument 1 passed to add() must be of the type integer, float given, called in /var/www/html/type-dealaration.php on line 12 and defined in /var/www/html/type-dealaration.php:5
Stack trace:
#0 /var/www/html/type-dealaration.php(12): add(1.2, 8.8)
#1 {main}
  thrown in /var/www/html/type-dealaration.php on line 5
```

使用类型声明可以使代码更加易于阅读，但也暴露出隐式类型转换的问题。

## 错误处理

在过去的`PHP`中，所有的`Error`其实都是`Exception`，在`PHP7`中将`Exception`和`Error`进行分开，两都去实现了`Throwable`接口。

现在设置强制约束类型，使用`TypeError`来捕获错误。

```php
<?php
declare(strict_types=1);

function add(int $a, int $b)
{
    return $a + $b;
}

try {
    echo add(1.2, 8.8) . "\n";
} catch(TypeError $e) {
     echo $e->getMessage() . "\n";
}

```

转出如下：

```bash
Argument 1 passed to add() must be of the type integer, float given, called in /var/www/html/type-dealaration-error.php on line 10
```

如果不加以捕获，收程序终止执行。

## 新的操作符号

> <=> 空间关系操作符：也叫做组合比较符号，可用于代替大于`>`和`<`

```php
<?php

echo 1 <=> 2; // -1
echo "\n";
echo 2 <=> 2; // 0
echo "\n";
echo 3 <=> 2; // 1
echo "\n";
```

通过使用`<=>`来判断数值大小。


> ?? Null合并符：二目运算符，若左操作数不为null，则返回左操作数，否则返回右操作数

在过去的使用中，常使用三目运算符来实现上述操作，如`$gender = $request->get('gender') == null ? 1 : 0`，现在有内置支持了

```php
<?php
$gender = null;

echo $request->get('gender') ?? 1; // 1  默认为男
```

还可以进行组合操作

```php
$last = $a ?? $b ?? $c ?? ...
```

最终遇到不为`null`的为止。

## 参考

* [哈希表-新实现](http://nikic.github.io/2014/12/22/PHPs-new-hashtable-implementation.html)
* [zval-新实现1](http://nikic.github.io/2015/05/05/Internal-value-representation-in-PHP-7-part-1.html)
* [zval-新实现2](http://nikic.github.io/2015/06/19/Internal-value-representation-in-PHP-7-part-2.html)
* [PHP7 new features](http://blog.teamtreehouse.com/5-new-features-php-7)
