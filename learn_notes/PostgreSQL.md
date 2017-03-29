# [查看最新版本，目前9.6](https://yum.postgresql.org/)
- yum -y install https://download.postgresql.org/pub/repos/yum/9.6/redhat/rhel-7-x86_64/pgdg-centos96-9.6-3.noarch.rpm
- yum -y install postgresql96-server postgresql96-contrib
- 初始化数据库
- /usr/pgsql-9.6/bin/postgresql96-setup initdb
- systemctl start postgresql-9.6.service
- systemctl enable postgresql-9.6.service
- yum -y install phpPgAdmin httpd
- systemctl start httpd
- systemctl enable httpd
# 修改密码
```sh
PostgreSQL 安装完成后，会建立一下‘postgres’用户，用于执行PostgreSQL，
vi /etc/passwd
数据库中也会建立一个'postgres'用户，默认密码为自动生成，需要在系统中改一下。
passwd postgres  为默认创建的系统用户设置个密码
cd ~postgres/
su  postgres  切换用户，执行后提示符会变为 '-bash-4.2$'
psql -U postgres 登录数据库，执行后提示符变为 'postgres=#'
\password postgres 设置及密码
create role uername login encrypted password '123456';  
alter role postgres with ENCRYPTED password '123456';  最后要有; 成功提示ALTER ROLE
alter user postgres WITH ENCRYPTED PASSWORD '123456'  

CREATE USER replica REPLICATION LOGIN ENCRYPTED PASSWORD 'yourpassword';
\q  退出数据库
```
# phpPgAdmin   http://ip/phpPgAdmin/
```sh
vi /etc/httpd/conf.d/phpPgAdmin.conf
        # Apache 2.4
        Require all granted
        #Require host example.com

        # Apache 2.2
        Order deny,allow
        Allow from all
        # Allow from .example.com

vi /etc/phpPgAdmin/config.inc.php
$conf['extra_login_security'] = false;
$conf['servers'][0]['host'] = '192.168.125.144';

systemctl restart httpd
```
# 开启远程访问
- vi /var/lib/pgsql/9.6/data/postgresql.conf
- 修改#listen_addresses = 'localhost'  为  listen_addresses='*'
- password_encryption=on
- ‘*’也可以改为任何你想开放的服务器IP
- vi /var/lib/pgsql/9.6/data/pg_hba.conf
- IPv4 local connections:
- host  all    all    192.168.125.1/24      trust   
- host  all    all    0.0.0.0/0    md5
# 主从配置
```sh
配置master
vi /var/lib/pgsql/9.6/data/postgresql.conf
listen_addresses = '*'          
wal_level = replica             # 网上很多教程设置为hot_standby，均为老版本的选项
max_wal_senders = 2                # 控制主库最多可以有多少个并发的standby数据库
wal_keep_segments = 64            # 配置保存wal日志的大小
synchronous_standby_names = '*' #synchronous_standby_names 这个参数对应着slave配置文件中的recovery.conf 中的primary_conninfo
log_connections = on             # 方便调试，下同
log_line_prefix = '%t [%p-%l] %q%u@%d '

vi /var/lib/pgsql/9.6/data/pg_hba.conf
host    replication     rep     IP_address_of_slave/32  md5

创建同步用户
CREATE USER replica REPLICATION LOGIN ENCRYPTED PASSWORD '123456';
\du 查看

配置Slave
vi /var/lib/pgsql/9.6/data/postgresql.conf
listen_addresses = '*'          
wal_level = replica             # 网上很多教程设置为hot_standby，均为老版本的选项
hot_standby = on
hot_standby_feedback = on
max_wal_senders = 2
wal_keep_segments = 64
log_connections = on             # 方便调试，下同
log_line_prefix = '%t [%p-%l] %q%u@%d '

vi /var/lib/pgsql/9.6/data/pg_hba.conf
host    replication     rep     IP_address_of_master/32  md5

主从都停掉postgresql
master运行同步命令，要密码，到底是哪个密码暂时不清楚
rsync -cva --inplace --exclude=*pg_xlog* /var/lib/pgsql/9.6/data/ IP_address_of_slave:/var/lib/pgsql/9.6/data/
启动时Slave报错了
```
# 防火墙
- setsebool -P httpd_can_network_connect_db 1
- firewall-cmd --permanent --zone=public --add-port=80/tcp
- firewall-cmd --permanent --zone=public --add-service=postgresql
# 启动
```sh
su postgres
/usr/pgsql-9.6/bin/pg_ctl -D /var/lib/pgsql/9.6/data/ -l logfile start

```

#### [xlgps](http://www.xlgps.com/article/343029.html)
#### [主从](http://www.jianshu.com/p/41bf119cac9a)
