# CentOS路由规则

## 打开80和22端口

```bash
/sbin/iptables -I INPUT -p tcp --dport 80 -j ACCEPT
/sbin/iptables -I INPUT -p tcp --dport 22 -j ACCEPT
```

## 保存、重启、关闭

```bash
/etc/init.d/iptables save
/etc/init.d/iptables restart
/etc/init.d/iptables 
```