# 常用且实用的小建议

node 下载下来的常常是 nodejs，使用下面命令将其链接

```bash
sudo ln -s `which nodejs` /usr/bin/node
```

npm 自动把模块和版本号添加到devdependencies部分

```bash
npm install xxx --save-dev
```

在 Docker 中，有时你想看下当前容器都挂载了哪些目录，可以使用下面的：

```bash
docker inspect --format='{{.Mounts}}' $containerID
```
