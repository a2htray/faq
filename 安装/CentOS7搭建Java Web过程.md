# CentOS 7 搭建 Java Web 过程

## 安装 Java 环境

```bash
yum install java-1.8.0-openjdk
yum install java-1.8.0-openjdk-devel
```

## 安装 Tomcat

```bash
groupadd tomcat
useradd -M -s /bin/nologin -g tomcat -d /opt/tomcat tomcat

# 这个链接最好自己去官网上看下，该文件是否存在
wget wget http://apache.mirrors.ionfish.org/tomcat/tomcat-8/v8.5.29/bin/apache-tomcat-8.5.29.tar.gz

mkdir /opt/tomcat
tar xvf apache-tomcat-8*tar.gz -C /opt/tomcat --strip-components=1
cd /opt/tomcat
chgrp -R tomcat /opt/tomcat
chmod -R g+r conf
chmod g+x conf
chown -R tomcat webapps/ work/ temp/ logs/
vi /etc/systemd/system/tomcat.service
```

将下面的文本粘贴进去

```text
# Systemd unit file for tomcat
[Unit]
Description=Apache Tomcat Web Application Container
After=syslog.target network.target

[Service]
Type=forking

Environment=JAVA_HOME=/usr/lib/jvm/jre
Environment=CATALINA_PID=/opt/tomcat/temp/tomcat.pid
Environment=CATALINA_HOME=/opt/tomcat
Environment=CATALINA_BASE=/opt/tomcat
Environment='CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC'
Environment='JAVA_OPTS=-Djava.awt.headless=true -Djava.security.egd=file:/dev/./urandom'

ExecStart=/opt/tomcat/bin/startup.sh
ExecStop=/bin/kill -15 $MAINPID

User=tomcat
Group=tomcat
UMask=0007
RestartSec=10
Restart=always

[Install]
WantedBy=multi-user.target
```

```bash
systemctl daemon-reload
systemctl start tomcat
systemctl enable tomcat
```

## 参考

* Java 安装 [https://www.digitalocean.com/community/tutorials/how-to-install-java-on-centos-and-fedora](https://www.digitalocean.com/community/tutorials/how-to-install-java-on-centos-and-fedora)

