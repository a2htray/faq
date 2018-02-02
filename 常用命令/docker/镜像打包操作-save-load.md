# 镜像打包操作save


* 将指定镜像打包成`tar`文件

```bash
docker save --output filename.tar image:tag
```

# load镜像操作

* 将指定`tar`包加载成镜像

```bash
docker load --input filename.tar
```
