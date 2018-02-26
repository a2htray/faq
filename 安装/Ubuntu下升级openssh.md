# Ubuntu下升级openssh

> 这个记录其实是接着之前升级[openssl](./Ubuntu下升级openssl.md)

```bash
cd /usr/src/
wget https://fastly.cdn.openbsd.org/pub/OpenBSD/OpenSSH/portable/openssh-7.6p1.tar.gz
tar -zxf openssh-7.6p1.tar.gz
cd openssh-7.6p1/
apt-get install zlib1g-dev # 没有这个会把ziblib not found
apt-get install libssl-dev
./configure --prefix=/usr --with-zlib --with-md5-passwords --with-ssl-dir=/usr/src/openssl-1.0.2n/ssl # 这个with-ssl-dir选项就是前文安装的openssl
make && make install
```
