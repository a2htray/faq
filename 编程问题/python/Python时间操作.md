# Python时间操作

## 将当前时间格式化

```python
t = time.strftime("%Y-%m-%d %H:%M:%S", time.localtime())
```

## 取一天起始时间和结束时间

```python
import datetime

print datetime.datetime.combine(datetime.datetime.now(), datetime.time.min)
print datetime.datetime.combine(datetime.datetime.now(), datetime.time.max)
```

## 计算两个日期的差

```python
# 例子1
import datetime

past = datetime.datetime.strptime('2014-02-14 21:32:12', '%Y-%m-%d %H:%M:%S')
now = datetime.datetime.now()

print (now - past).days

# 例子2
import datetime

past = datetime.datetime.strptime('2014-02-14 21:32:12', '%Y-%m-%d %H:%M:%S').date()
now = datetime.datetime.now().date()

print (now - past).days
```

