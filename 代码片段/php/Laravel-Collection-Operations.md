# Laravel Collection Operations

## 使用回调函数按对象的某一个方法返回值排序

```php
Course::with('speakerInfo')->where('isclose', '=', 1)->get()
    ->sortBy(function ($value, $key) {
        return $value->getTotal();
}, SORT_DESC, true);
```