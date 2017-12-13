# 简单make命令的工作流程

```c
make helloworld
```

## 目的

创建一个`helloworld`的文件

## 流程

* 文件`helloworld`是否存在？
* 如果没有。看有没有以`helloworld`开头的文件
* 如果找到`helloworld`，执行`cc helloworld.c -o helloworld`

## CFLAGS

设置环境变量，修改`cc`时的选项

```c
CFLAGS="-Wall" make helloworld 
# 执行cc时，形式如"cc -Wall helloworld.c -o helloworld"
```