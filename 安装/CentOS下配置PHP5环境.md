# CentOS 7 下 PHP5 环境配置

## Nginx 安装

```bash
yum install epel-release
yum install nginx
```

## php 安装

```bash
yum install php php-fpm
```

### 修改配置文件

* 修改`/etc/php-fpm.d/www.conf`下中

```fpm
- user = apache
+ user = nginx
- group = apache
+ group = nginx
```

* 在`/etc/nginx/nginx.conf` `server块` 中加入

```nginx
location ~ \.php$ {
    fastcgi_pass   127.0.0.1:9000;
    fastcgi_index  index.php;
    fastcgi_param  SCRIPT_FILENAME /usr/share/nginx/html$fastcgi_script_name;
    include        fastcgi_params;
}
```

### 安装 composer

```bash
curl -sS https://getcomposer.org/installer | php
mv composer.phar /usr/local/bin/composer
```

## 安装 redis 及 php-redis

```bash
yum install redis php-redis
```

### 配置 php.ini

通过`phpinfo()`查看`php.ini`路径，修改`/etc/php.ini`

```php
+ extension=redis.so
```

### 永久关闭 selinux

`vi /etc/selinux/config`

```bash
- SELINUX=enforcing
+ SELINUX=disalbed
```

进行重启即可以，不然会报`RedisException Redis server went away `

## 安装 MySQL Server 及 php-mysql

```bash
wget https://dev.mysql.com/get/mysql57-community-release-el7-9.noarch.rpm
rpm -ivh mysql57-community-release-el7-9.noarch.rpm
yum install mysql-server
systemctl start mysqld
grep 'temporary password' /var/log/mysqld.log # 查看临时生成的密码
mysql_secure_installation # 进行数据库相关设置
```

## 安装 Samba Server

实现 Windows 与 VirtualBox 中的 CentOS7 交互

```bash
yum install samba* -y
```

打到配置文件`/etc/samba/smb.conf`，底部加入

```bash
[share]
        # 这个路径自行选择
        path = /usr/share/nginx/html/sf-project 
        writable = yes
        browsable = yes
        guest ok = yes
        guest only = yes
        create mode = 0777
        directory mode = 0777
```

为`root`用户创建samba密码

```bash
pdbedit -a root
systemctl restart smb
systemctl restart nmb
```

## 重启服务

```bash
service nginx restart
service php-fpm restart
```

## 结语

环境基本配置完成，可以开始做一些测试了。

## 参考

* Nginx 安装 [https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-centos-7](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-centos-7)
* Nginx 配置 php-fpm 配置 [https://qiita.com/toshihirock/items/77835f83f679423874ea](https://qiita.com/toshihirock/items/77835f83f679423874ea)
* redis 配置 [https://www.jianshu.com/p/888901a7f642](https://www.jianshu.com/p/888901a7f642)
* 关闭 SELINUX [http://www.dear521.com/home/blog/index/id/37.html](http://www.dear521.com/home/blog/index/id/37.html)
* 安装 MySQL [https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-centos-7](https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-centos-7)
* Samba Server [https://www.unixmen.com/install-configure-samba-server-centos-7/](https://www.unixmen.com/install-configure-samba-server-centos-7/)