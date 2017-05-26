# CentOS7-MySQL安装
```sh
https://dev.mysql.com/downloads/repo/yum/
rpm -Uvh https://dev.mysql.com/get/mysql57-community-release-el7-9.noarch.rpm
yum list | grep mysql
yum -y install mysql-community-server
```
```sh
/usr/bin/systemctl start mysqld
grep "password" /var/log/mysqld.log 使用YUM安装并启动MySQL服务后，MySQL进程会自动在进程日志中打印root用户的初始密码
/usr/bin/systemctl enable mysqld

```
#  拒绝登录
```sh
Access denied for user 'root'@'localhost' (using password: YES)
mysqld stop 
mysqld_safe --user=mysql --skip-grant-tables --skip-networking & 
mysql -u root mysql 

```
# 忘记密码
```sh
sudo vi /etc/my.cnf
[mysqld] 
skip-grant-tables 
validate-password=OFF 
user=root

5.7mysql> update user set authentication_string=PASSWORD('newpass') where User='root';
5.6mysql> update user set password=PASSWORD('newpass') where User='root';

```
# 创建
```sh
create database testDB default charset utf8 collate utf8_general_ci;
CREATE USER 'test'@'%' IDENTIFIED BY '123456-Abc';  //不要偷懒去修改密码规则
alter user 'test'@'%' password expire never;
alter user 'test'@'%' password account lock/unloc;

grant select,insert,update,delete on testDB.* to 'test'@'%';
flush privileges; 
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
firewall-cmd --permanent --zone=public --add-port=3306/tcp
firewall-cmd  --reload
```
```sh
//添加服务用户
useradd -r -M -s /sbin/nologin mysql
//更改mysql安装目录属主 属组
cd /usr/local/mysql5.6/
chown -R mysql:mysql /usr/local/mysql5.6/
 
//创建
 
//配置my.cnf，从源码包中复制my.cnf文件
cd /usr/local/mysql5.6
cp my.cnf /etc/
 
//根据配置文件 配置各log sock pid目录的属主 属组
chown mysql:mysql ****** (省略)
 
//初始化数据库
cd /usr/local/mysql5.6/
scripts/mysql_install_db --user=mysql
 
//配置环境变量，把mysql/bin目录添加到PATH变量中
echo 'export PATH=$PATH:/usr/local/mysql5.6/bin' >> /etc/profile   &&   source /etc/profile
 
//试运行
mysqld_safe
 
//如果运行无报错的话，使用mysql客户端程序连接
mysql --protocol=tcp -h localhost -u root
 
//配置CentOS 7上的mysqld服务文件
cd /root/mysql-5.6.31/packaging/rpm-fedora/
cp mysqld.service /usr/lib/systemd/system
systemctl list-unit-files -t service | grep mysql
cd /etc/systemd/system/multi-user.target.wants/
ln -s /usr/lib/systemd/system/mysqld.service ./
 
//更改mysqld.service 默认配置
//可以把mysqld.service中的 ExecStartPre 和 ExecStartPost注释掉
//更改 ExecStart 项配置，路径必须为绝对路径，ExecStart=/usr/local/mysql5.6/bin/mysqld_safe
 
//测试mysqld.service 是否能够正常启动、重启、停止服务
systemctl daemon-reload
systemctl start mysqld.service
```
# vi /etc/my.cnf
```sh
# For advice on how to change settings please see
# http://dev.mysql.com/doc/refman/5.6/en/server-configuration-defaults.html
 
[mysqld]
#
# Remove leading # and set to the amount of RAM for the most important data
# cache in MySQL. Start at 70% of total RAM for dedicated server, else 10%.
# innodb_buffer_pool_size = 128M
#
# Remove leading # to turn on a very important data integrity option: logging
# changes to the binary log between backups.
# log_bin
#
# Remove leading # to set options mainly useful for reporting servers.
# The server defaults are faster for transactions and fast SELECTs.
# Adjust sizes as needed, experiment to find the optimal values.
# join_buffer_size = 128M
# sort_buffer_size = 2M
# read_rnd_buffer_size = 2M
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
 
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0
 
# Recommended in standard MySQL setup
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
 
[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
 
[client]
host=localhost
user=root
protocol=socket
socket=/var/lib/mysql/mysql.sock
```
```sh
#
# Simple MySQL systemd service file
#
# systemd supports lots of fancy features, look here (and linked docs) for a full list:
#   http://www.freedesktop.org/software/systemd/man/systemd.exec.html
#
# Note: this file ( /usr/lib/systemd/system/mysqld.service )
# will be overwritten on package upgrade, please copy the file to
#
#  /etc/systemd/system/mysqld.service
#
# to make needed changes.
#
# systemd-delta can be used to check differences between the two mysqld.service files.
#
 
[Unit]
Description=MySQL Community Server
After=network.target
After=syslog.target
 
[Install]
WantedBy=multi-user.target
Alias=mysql.service
 
[Service]
User=mysql
Group=mysql
Type=simple
# PIDFile=/var/run/mysqld.pid
 
# Execute pre and post scripts as root
PermissionsStartOnly=true
 
# Needed to create system tables etc.
ExecStartPre=-/usr/bin/mkdir /var/run/mysqld/
ExecStartPre=/usr/bin/chown mysql:mysql /var/run/mysqld/
 
# Start main service
ExecStart=/usr/local/mysql5.6/bin/mysqld_safe
ExecReload=/bin/kill -HUP $MAINPID
# Don't signal startup success before a ping works
# ExecStartPost=/usr/local/mysql5.6/bin/mysql-systemd-start post
 
# Give up if ping don't get an answer
TimeoutSec=60
 
Restart=on-failure
PrivateTmp=false
```
