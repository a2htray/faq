# Ubuntu下安装PhpStrom

```bash
# 安装java jdk
sudo apt-get purge openjdk*
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java8-installer oracle-java8-set-default

# 安装PhpStorm
wget http://download-cf.jetbrains.com/webide/PhpStorm-2017.1.4.ta‌​r.gz
tar -xvf PhpStorm-2017.1.4.ta‌​r.gz
cd PhpStorm-171.4694.2/bin/
./phpstorm.sh
```

[问题原文](https://askubuntu.com/questions/474151/how-to-install-phpstorm-in-ubuntu-14-04-lts)