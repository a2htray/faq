# postgresql的备份与恢复

使用`pg_dump`用于导出数据，生成备份文件

```bash
pg_dump -h localhost -U test_user test_db > test_db.bak
```

使用`psql`命令还原

```
psql -h localhost -U test_user -d test_db < test_db.bak
```