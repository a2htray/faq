# Ubuntu下升级openssl

**亲测有效**

**[教程](https://www.linuxhelp.com/how-to-install-and-update-openssl-on-ubuntu-16-04/)**

```bash
cd /usr/src
wget https://www.openssl.org/source/openssl-1.0.2-latest.tar.gz
apt-get install -y make gcc
tar -zxf openssl-1.0.2-latest.tar.gz
cd openss-version # 这里根据实际情况修改
./config
make
make install

# 如果旧的版本需要保留，使用mv作备份
mv /usr/bin/openssl /root/
ln -s /usr/local/ssl/bin/openssl /usr/bin/openssl
```
