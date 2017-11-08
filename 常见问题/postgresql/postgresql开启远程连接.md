# postgresql开启远程连接

> 需要修改postgresql的配置文件，`postgresql.conf`, `pg_hba.conf`，然后通过`pg_ctl`命令重启服务

1. 在`postgresql.conf`文件中，找到

```bash
# listen_addresses = 'localhost'  # what IP address(es) to listen on;
```

修改成如下

```bash
listen_addresses = '*'  # what IP address(es) to listen on;
```

2. 添加一条访问路由，在`pg_hba.conf`的末行上

```bash
host    all             all             0.0.0.0/0               md5
```

3. 通过`pg_ctl`重启服务

> 以下是本机执行的命令，根据个人实际情况修改

```bash
sudo -u postgres /usr/lib/postgresql/9.5/bin/postgres -D /var/lib/pgsql/data restart
```