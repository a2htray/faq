# mysql日期操作语句

## 获取前n天的记录

使用`DATE_SUB`，将当前天减去`n`天，再与其他字段进行比较

```sql
SELECT `code` FROM `recommend_stocks`
 WHERE
 DATE_SUB(DATE('2017-11-27', INTERVAL n DAY)) < DATE(`created_at`);
```

上述的`n`填写适当的值

## 计算两个日期相差的天数

```sql
DATEDIFF(CURRENT(), `recommend_stocks`.`created_at`)
```