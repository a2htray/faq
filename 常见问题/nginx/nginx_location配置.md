# nginx location 配置

## 匹配目录常用`^~`

使用`^~`常用于匹配目录，匹配成功并停止匹配其他项

```bash
location ^~ /images/ {
  # 匹配任何以 /images/ 开始的请求，并停止匹配 其它location
}
```

