# Upgrade php 5.4 to php 7

## CentOS7

```bash
yum install https://centos7.iuscommunity.org/ius-release.rpm
yum remove php-common mod_php php-cli
yum update
yum install php70u php70u-fpm php70u-json php70u-mbstring php70u-pdo php70u-mysqlnd php70u-opcache php70u-xml php70u-gd php70u-devel php70u-mysql 
```

## 参考

* CentOS 7 [Upgrade to PHP 7](https://www.1and1.com/cloud-community/learn/application/php/upgrade-php-from-54-to-70-on-a-centos-7-11-cloud-server/)