# ubuntu下时区操作

## 修改时区

```bash
echo "Asia/Shanghai" > /etc/timezone && dpkg-reconfigure -f noninteractive tzdata
```