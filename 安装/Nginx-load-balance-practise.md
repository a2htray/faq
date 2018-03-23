# 使用 Docker 测试 Ngnix 负载均衡


```bash
docker run --name web-worker-01 -d nginx
docker run --name web-worker-02 -d nginx
docker run --name web-scheduler \
--link web-worker-01:web-worker-01 \
--link web-worker-02:web-worker-02 -p 90:80 -d nginx
```

使用`docker run`启动三个容器服务，两台`worker`，对外提供服务的命名为`web-scheduler`，并使用`--link`选项，使用在`scheduler`中可以访问到两台`worker`。如下图所示：

![container profile](./images/微信截图_20180323170625.png)

使用`docker exec`进行`scheduler`容器

```bash
docker exec -it web-scheduler /bin/bash
```

编辑`/etc/nginx/nginx.conf`和`/etc/nginx/conf.d/default.conf`这两个文件，默认的`nginx`镜像中没集成`vim`工具，可自行使用`apt-get install`安装。

在`http`块中加入`upstream`块，修改`location`块进行转发

修改如下:

```nginx
# /etc/nginx/nginx.conf

http {
    upstream backend {
        server web-worker-01;
        server web-worker-02;
    }
}

######################################

# /etc/nginx/conf.d/default.conf

server {
    location / {
        proxy_pass http://backend;
    }
}
```

作为测试，直接修改两台`worker`中的主页的内容，实际情况中，`worker`的代码应保持一致。




## 参考

*  [Ngnix Offical Load Balance Tutorial](http://nginx.org/en/docs/http/load_balancing.html)