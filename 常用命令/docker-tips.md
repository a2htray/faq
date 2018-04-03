# Docker 实用命令


## 日志

**打印容器输出尾部50行**

```bash
docker logs --tail 50 --follow --timestamps docker_web_1
```

## 删除

**删除镜像名为<none>的镜像**

```bash
docker rmi -f $(docker images | grep none | awk '{print $3}')
```

## 查看

