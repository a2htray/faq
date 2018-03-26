# Nginx 的11个请求阶段

Ngnix 处理一个请求共有好几个阶段，每个阶段都使用一个特殊的常量进行标记。

<!-- more -->

下面是各个阶段下都需要执行的动作：

1. `NGX_HTTP_POST_READ_PHASE`：请求头部处理
2. `NGX_HTTP_SERVER_REWRITE_PHASE`：进行`URL`解析并映射到虚拟服务器层
3. `NGX_HTTP_FIND_CONFIG_PHASE`：查找对应的`location`块 ☹
4. `NGX_HTTP_REWRITE_PHASE`：进行路由重写
5. `NGX_HTTP_POST_REWRITE_PHASE`：继续处理`URL`请求的一些后期工作 ☹
6. `NGX_HTTP_PREACCESS_PHASE`：预处理检查访问权限
7. `NGX_HTTP_ACCESS_PHASE`：检查访问权限
8. `NGX_HTTP_POST_ACCESS_PHASE`：权限未通过，返回拒绝错误码 ☹
9. `NGX_HTTP_TRY_FILES_PHASE`：处理`try_files`块信息，为访问静态文件资源而设置 ☹
10. `NGX_HTTP_CONTENT_PHASE`：内容生成
11. `NGX_HTTP_LOG_PHASE`：日志记录

解析`URL` -> 找路由 -> 权限检查 -> 处理静态文件 -> 生成内容 -> 记录目录


## 参考

* [英文 10个请求阶段](http://www.nginxguts.com/2011/01/phases/)
* [HTTP请求的11个处理阶段](https://blog.csdn.net/nestler/article/details/30805395)