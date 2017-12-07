# Ubuntu下crontab定时任务

如果没有预先安装cron，执行下面命令

```bash
apt-get update
apt-get install -y cron
```

安装好后，在目录`/etc/`包含以下信息

<div align="center">

![](/安装/images/cron服务目录信息.png)

</div>

## crontab文件格式

分 时 日 月 星期 要运行的命令

* 第1列分钟0～59
* 第2列小时0～23（0表示子夜）
* 第3列日1～31
* 第4列月1～12
* 第5列星期0～7（0和7表示星期天）
* 第6列要运行的命令

### 周一到周五每天下午5:00执行

```bash
0 17 * * 1-5 scrapy crawl gupiao_update_earning
```
