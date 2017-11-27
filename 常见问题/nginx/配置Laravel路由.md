# 配置Laravel路由

需要在配置文件中加入

```bash
location / {
    try_files $uri $uri/ /index.php?$query_string;
}
```

因为上面的配置写错，导致我一直获取不到正确的`GET`参数