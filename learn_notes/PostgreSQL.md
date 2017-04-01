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

CREATE USER replica REPLICATION LOGIN ENCRYPTED PASSWORD '123456';
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
配置master 192.168.125.147
创建同步用户
psql
CREATE USER replica REPLICATION LOGIN ENCRYPTED PASSWORD '123456';
\du 查看

vi /var/lib/pgsql/9.6/data/postgresql.conf
listen_addresses = '*'          
wal_level = replica             # 网上很多教程设置为hot_standby，均为老版本的选项
max_wal_senders = 2                # 控制主库最多可以有多少个并发的standby数据库,差不多有几个从，就设置多少
wal_keep_segments = 64            # 配置保存wal日志的大小
synchronous_standby_names = '*' #synchronous_standby_names 这个参数对应着slave配置文件中的recovery.conf 中的primary_conninfo
log_connections = on             # 方便调试，下同
log_line_prefix = '%t [%p-%l] %q%u@%d '
wal_send_timeout = 60s 
max_connections = 512 #从库的 max_connections要大于主库
archive_mode = on #允许归档 



vi /var/lib/pgsql/9.6/data/pg_hba.conf
host    replication     replica     192.168.125.148/32  md5

配置Slave  192.168.125.148
rm -rf /var/lib/pgsql/9.6/data/*  #开始没有启动从数据库，这一步可以省略 
pg_basebackup -h ip-of-master -U repl -D /var/lib/pgsql/9.6/data -X stream -P
cp /usr/pgsql-9.6/share/recovery.conf.sample /var/lib/pgsql/9.6/data/recovery.conf

vi /var/lib/pgsql/9.6/data/recovery.conf
standby_mode = on
primary_conninfo = 'host=192.168.125.147 port=5432 user=replica password=123456'
trigger_file = '/var/lib/pgsql/9.6/data/trigger.kenyon'    #主从切换时后的触发文件
recovery_target_timeline = 'latest'

vi /var/lib/pgsql/9.6/data/postgresql.conf
listen_addresses = '*'          
wal_level = replica             # 网上很多教程设置为hot_standby，均为老版本的选项
max_wal_senders = 2
wal_keep_segments = 64
log_connections = on             # 方便调试，下同
log_line_prefix = '%t [%p-%l] %q%u@%d '
max_connections = 1000 #一般从的最大链接要大于主的。 
hot_standby = on #说明这台机器不仅仅用于数据归档，也用于查询 
max_standby_streaming_delay = 30s 
wal_receiver_status_interval = 10s #多久向主报告一次从的状态。 
hot_standby_feedback = on #如果有错误的数据复制，是否向主进行范例




master库
psql
select client_addr,sync_state from pg_stat_replication;
select * from pg_stat_replication;

netstat -lntup|grep 5432 && ps -ef|grep postmaster
ps -ef | grep postgres  可以看到wal sender/receiver process

CREATE TABLE rep_test (test varchar(40));
INSERT INTO rep_test VALUES ('data one');
SELECT * FROM rep_test;




cd /var/lib/pgsql/9.6/data/trigger.kenyon
touch trigger.kenyon  切换主从
cd /usr/pgsql-9.6/bin
./pg_controldata



下面有问题
vi /var/lib/pgsql/9.6/data/pg_hba.conf
host    replication     rep     IP_address_of_master/32  md5

主从都停掉postgresql
master运行同步命令，要密码，到底是哪个密码暂时不清楚
rsync -cva --inplace --exclude=*pg_xlog* /var/lib/pgsql/9.6/data/ IP_address_of_slave:/var/lib/pgsql/9.6/data/
启动时Slave报错了
psql -x -c "select * from pg_stat_replication;"
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
