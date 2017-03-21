# CentOS7-MySQL安装
```sh
https://dev.mysql.com/downloads/repo/yum/
rpm -Uvh https://dev.mysql.com/get/mysql57-community-release-el7-9.noarch.rpm
yum list | grep mysql
yum -y install mysql-community-embedded

```
```sh
/usr/bin/systemctl start mysqld
/usr/bin/systemctl enable mysqld

```
```sh
/usr/bin/mysql_secure_installation
Set root password? [Y/n] Y
Remove anonymous users? [Y/n] Y
Disallow root login remotely? [Y/n] Y
Remove test database and access to it? [Y/n] Y
Reload privilege tables now? [Y/n] Y
```
```sh
firewall-cmd --permanent --zone=trusted --add-source=192.0.2.10/32
firewall-cmd --permanent --zone=trusted --add-port=3306/tcp
firewall-cmd  --reload
```


