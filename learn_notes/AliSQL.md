# [docker](https://github.com/alibaba/AliSQL/issues/14)
# [dockerhub](https://hub.docker.com/r/tekintian/alisql/)
# docker方式运行
```sh
yum -y install deltarpm
yum -y install docker
systemctl start docker
systemctl enable docker
docker pull registry.cn-hangzhou.aliyuncs.com/acs-sample/alisql:latest

firewall-cmd --permanent --zone=public --add-port=6657/tcp
systemctl restart firewalld

下面命令将mysqld换为/bin/bash也可跑起来但是无法连接数据库，貌似要安装mysql？
我安装的是mysql5.7但是alisql好像是5.6
docker run -it -p 6657:3306 -v /etc/my.cnf:/etc/my.cnf -e MYSQL_ROOT_PASSWORD=hello1234 --name alisql --restart=always -d registry.cn-hangzhou.aliyuncs.com/acs-sample/alisql:latest mysqld   

修改docker的root用户密码
docker exec -it alisql bash
mysql -u root -p mysql
update user set password=PASSWORD('12345678') where User='root'; //可以判断alisql不是5.7
\q
exit
以后  docker start alisql  即可
docker inspect alisql
docker ps -a
docker stop  alisql
docker rm alisql

COMMAND_FAILED: '/sbin/iptables -t nat -A Docker -p tcp -d 0/0 --dport 8111 -j DNAT --to-destination 172.17.0.6:8111 ! -i docker0' failed: iptables: No chain/target/match by that name.

pkill docker
iptables -t nat -F
ifconfig docker0 down
brctl delbr docker0
systemctl restart docker

/var/lib/docker/containers
```
```sh
[mysqld]
basedir=/data/alisql/data
datadir=/data/alisql/data
socket=/data/alisql/data/mysql.sock
symbolic-links=0

[mysqld_safe]
log-error=/data/alisql/log/mysqld.log
pid-file=/data/alisql/log/mysqld.pid
[client]
socket=/data/alisql/log/mysql.sock
```
# 若已安装要先卸载
- yum remove mysql
- rm -rf /var/lib/mysql
- rm /etc/my.cnf
- 查看是否还存在mysql软件
- rpm -qa|grep mysql
- 若存在，则继续
- yum –y remove mysql-*
# 安装环境
- yum -y install centos-release-scl 
- yum -y install devtoolset-4-gcc-c++ devtoolset-4-gcc
- yum -y install cmake Git
- yum -y install ncurses-devel openssl-devel bison
- scl enable devtoolset-4 bash
- yum -y install  gcc gcc-c++ make  ncurses  bison-devel zip unzip 这个不装
# 安装软件包
- mkdir /AliSQL 安装包保存位置
- cd /AliSQL
- wget https://github.com/alibaba/AliSQL/archive/master.zip
- unzip master.zip 
- cd AliSQL-master/source_downloads
- wget -o gmock-1.6.0.zip https://github.com/google/googlemock/archive/master.zip
# 关闭THP
- TokuDB 引擎的启动是不允许开启 THP（透明大页）的，否则就会启动失败，而且数据库类应用确实都不适合开启 THP
- echo never > /sys/kernel/mm/transparent_hugepage/enabled
- echo never > /sys/kernel/mm/transparent_hugepage/defrag
- 运行上面的两个语句，并将他们写入 /etc/rc.local
# 添加用户(方式一)
- groupadd mysql
- useradd -g mysql -s /sbin/nologin mysql
- ls -laF /usr/local/alisql 
- cd  /usr/local/alisql 
- chown -R mysql:mysql . 
- cd /data/alisqldb
- chown -R mysql:mysql . 
---
- useradd -s /sbin/nologin -M mysql
- mkdir -p /home/mysql/{data,logs,tmp} 
- chown -R mysql: /home/mysql/
---
- groupadd mysql
- useradd -g mysql mysql
- mkdir -p /usr/local/alisql
- mkdir -p /var/lib/alisql
- chown mysql.mysql -R /var/lib/alisql
- chmod +w /usr/local/alisql
- chown -R mysql:mysql /usr/local/alisql
- ln -s /usr/local/alisql/lib/libmysqlclient.so.18 /usr/lib/libmysqlclient.so.18
- cp support-files/my-default.cnf /etc/my.cnf
- cp support-files/mysql.server /etc/init.d/mysqld
- chmod a+x /etc/init.d/mysqld

# 编译安装/AliSQL/AliSQL-master
- cmake .  -DCMAKE_BUILD_TYPE="Release"      -DCMAKE_INSTALL_PREFIX="/usr/local/alisql"  -DWITH_EMBEDDED_SERVER=0              -DWITH_EXTRA_CHARSETS=all             -DWITH_MYISAM_STORAGE_ENGINE=1        -DWITH_INNOBASE_STORAGE_ENGINE=1      -DWITH_PARTITION_STORAGE_ENGINE=1     -DWITH_CSV_STORAGE_ENGINE=1           -DWITH_ARCHIVE_STORAGE_ENGINE=1       -DWITH_BLACKHOLE_STORAGE_ENGINE=1     -DWITH_FEDERATED_STORAGE_ENGINE=1     -DWITH_PERFSCHEMA_STORAGE_ENGINE=1    -DWITH_TOKUDB_STORAGE_ENGINE=1

- cmake -DMYSQL_USER=mysql -DCMAKE_INSTALL_PREFIX=/usr/local/alisql -DSYSCONFDIR=/usr/local/alisql -DMYSQL_UNIX_ADDR=/tmp/mysql.sock -DEXTRA_CHARSETS=all -DDEFAULT_CHARSET=utf8 -DDEFAULT_COLLATION=utf8_general_ci -DWITH_INNOBASE_STORAGE_ENGINE=1 -DWITH_MYISAM_STORAGE_ENGINE=1 -DWITH_ARCHIVE_STORAGE_ENGINE=0 -DWITH_MEMORY_STORAGE_ENGINE=0 -DENABLED_LOCAL_INFILE=1 -DWITH_EMBEDDED_SERVER=1  -DENABLE_DOWNLOADS=1 -DWITH_READLINE=1 -DWITH_DEBUG=0

- cmake . -DCMAKE_INSTALL_PREFIX=/usr/local/alisql -DMYSQL_TCP_PORT=3306 -DMYSQL_DATADIR=/var/lib/alisql -DSYSCONFDIR=/etc -DWITH_INNOBASE_STORAGE_ENGINE=1 -DWITH_PARTITION_STORAGE_ENGINE=1 -DWITH_FEDERATED_STORAGE_ENGINE=1 -DWITH_BLACKHOLE_STORAGE_ENGINE=1 -DWITH_TOKUDB_STORAGE_ENGINE=1 -DWITH_MYISAM_STORAGE_ENGINE=1 -DMYSQL_UNIX_ADDR=/tmp/mysql.sock -DWITH_EMBEDDED_SERVER=0 -DENABLE_DTRACE=0 -DENABLED_LOCAL_INFILE=1 -DDEFAULT_CHARSET=utf8mb4 -DDEFAULT_COLLATION=utf8mb4_general_ci -DCMAKE_EXE_LINKER_FLAGS="-ljemalloc" -DWITH_SAFEMALLOC=OFF -DEXTRA_CHARSETS=all

- make && make install
- make -j4 && make install 多CPU支持，数字一般比CPU数量多1即可
# 重新编译
```sh
 rm CMakeCache.txt 
 ```
# 报错
```sh
make[2]: *** [sql/CMakeFiles/sql.dir/sql_yacc.cc.o] Error 1
make[1]: *** [sql/CMakeFiles/sql.dir/all] Error 2
make: *** [all] Error 2
这个是gcc的版本问题，
使用yum mysql-server后然后remove掉mysql-server
yum -y install mysql-server
yum -y remove  mysql-server
```
# 初始化/usr/local/alisql，
- find / -name my_print_defaults
- /usr/local/alisql/bin/my_print_defaults  basedir就是/usr/local/alisql
- cd /home/zhangtao/AliSQL-master/scripts
- ./mysql_install_db --user=mysql --basedir=/usr/local/alisql --datadir=/data/alisqldb 对应添加用户方式一，数据目录
- 上句若提示无权限就进入压缩包的解压路径添加允许权限
- cd /AliSQL/AliSQL-master/scripts
- chmod +x mysql_install_db
- cp support-files/mysql.server /etc/init.d/mysqld
- cp support-files/my-default.cnf /etc/my.cnf
- vi /etc/my.cnf
- datadir = /data/alisqldb    对应添加用户方式一，数据目录
# 启动
- service mysqld start   即可启动
- chmod +x support-files/mysql.server
- support-files/mysql.server start
- ls -laF /opt/alisql
- chown -R mysql 
- cp /var/lib/mysql.sock /var/lib/mysql.sock_bkp
- rm -rf /var/lib/mysql.sock 
- chkconfig --add mysqld; chkconfig mysqld on  开机启动
# 配置数据库
```sh
2.配置数据库

[root@iZj6cich5sl4bw9f2e2erzZ cxt]# cd /usr/local/alisql   

[root@iZj6cich5sl4bw9f2e2erzZ alisql]# cp support-files/mysql.server /etc/init.d/mysqld

[root@iZj6cich5sl4bw9f2e2erzZ alisql]# cp support-files/my-default.cnf/etc/my.cnf

[root@iZj6cich5sl4bw9f2e2erzZ alisql]# service mysqld start

[root@iZj6cich5sl4bw9f2e2erzZ alisql]# chkconfig --level 35 mysqld on

2.       设置数据库root的密码

[root@iZj6cich5sl4bw9f2e2erzZ alisql]# cd /opt/alisql

[root@iZj6cich5sl4bw9f2e2erzZ alisql]# ./bin/mysqladmin -u root password'root123!@#'

登入数据库，允许root用户远程连接

[root@iZj6cich5sl4bw9f2e2erzZ alisql]# mysql -u root -p

mysql> use mysql;

mysql> update user set host='%' where user = 'root';

mysql> flush privileges;

mysql> quit
```
# 配置文件，附Large
vi /usr/local/alisql/my.cnf
```sh
[mysqld]
socket = /home/mysql/mysql.sock 
datadir = /home/mysql/data 
tmpdir = /home/mysql/tmp
port = 3306 
backlog = 3000 
character_set_server = utf8 
max_connect_errors = 100 
maxconnections = 16050 
max_user_connections = 16050 
max_heap_table_size = 64M 
max_allowed_packet = 1024M 
max_binlog_size = 500M 
threadstack = 256K 
interactivetimeout = 7200 
waittimeout = 86400 
sort_buffer_size = 848KB 
read_buffer_size = 848KB 
read_rnd_buffer_size = 432KB 
join_buffer_size = 432KB 
net_buffer_length = 16K 
thread_cache_size = 100 
ft_min_word_len = 4 
transactionisolation = READ-COMMITTED 
tmp_table_size = 262144 
table_open_cache = 1024 
skip_name_resolve 
core-file 
lower_case_table_names = 1

log_bin_trust_function_creators = 1 
log-bin = /home/mysql/logs/mysql-bin.log 
log-bin-index = /home/mysql/data/master-log-bin.index 
log-error = /home/mysql/mysql-error.log 
relay-log = /home/mysql/data/slave-relay.log 
relay-log-info-file = /home/mysql/data/slave-relay-log.info 
relay-log-index = /home/mysql/data/slave-relay-log.index 
master-info-file = /home/mysql/data/master.info

log-slave-updates = 1 
binlog_cache_size = 2048KB 
syncbinlog = 1000 
logwarnings 
slow_query_log_file = /home/mysql/logs/slowquery.log 
slow_query_log = 1 
logoutput = TABLE 
long_query_time = 3 
binlogformat = ROW 
serverid = 8075 
auto_increment_increment = 1 
auto_increment_offset = 1 
slave_net_timeout = 60 
key_buffer_size = 16M 
bulk_insert_buffer_size = 4M 
myisam_sort_buffer_size = 262144 
myisam_max_sort_file_size = 2048K 
myisam_repair_threads = 1 
myisam_recover_options = FORCE

innodb_data_home_dir = /home/mysql/data 
innodb_log_group_home_dir = /home/mysql/data 
innodb_additional_mem_pool_size = 2097152 
innodb_buffer_pool_size = 52429M 
innodb_data_file_path = ibdata1:200M:autoextend 
innodb_file_per_table 
innodb_file_io_threads = 4 
innodb_flush_log_at_trx_commit = 2 
innodb_log_buffer_size = 8M 
innodb_log_file_size = 1500M 
innodb_log_files_in_group = 2 
innodb_max_dirty_pages_pct = 75 
innodb_flush_method = ODIRECT 
innodb_lock_wait_timeout = 5000 
innodbdoublewrite = 1 
innodb_rollback_on_timeout = OFF 
innodb_autoinc_lock_mode = 1 
innodb_read_io_threads = 4 
innodb_write_io_threads = 4 
innodb_io_capacity = 2000 
innodb_purge_threads = 1

master_info_repository = TABLE 
relay_log_info_repository = TABLE 
query_cache_type = 0 
concurrentinsert = 1 
query_cache_limit = 1048576 
query_cache_min_res_unit = 1K 
log-slow-admin-statements 
innodb_stats_on_metadata = OFF 
innodb_file_format = Barracuda 
innodb_read_ahead = 0 
innodb_thread_concurrency = 0 
innodb_sync_spin_loops = 100 
innodb_spin_wait_delay = 30 
default_storage_engine = InnoDB 
innodb_stats_sample_pages = 8 
open_files_limit = 65535 
gtidmode = ON 
loose_rds-anonymous-in-gtid-out-enable = 1 
enforce-gtid-consistency = 1 
loose_opt_rds_enable_show_slave_lag = on 
loose_performance_schema = off 
loose_innodb_rds_buffer_pool_file_del = ON 
loose_binlog_order_commits = OFF 
innodb_ft_max_token_size = 84 
log_bin_use_v1_row_events = 1 
loose_innodb_rds_autoinc_persistent_interval = 1

delay_key_write = ON 
key_cache_division_limit = 100 
innodb_old_blocks_pct = 37 
loose_rds_gtid_precommit = ON 
loose_implicit_primary_key = 1 
ft_query_expansion_limit = 20 
loose_rds_binlog_group_commit_sync_no_delay_count = 0 
loose_tokudb_checkpointing_period = 60 
loose_thread_pool_stall_limit = 30 
loose_innodb_log_compressed_pages = OFF 
initconnect = '' 
innodb_print_all_deadlocks = OFF 
delayed_insert_timeout = 300 
connecttimeout = 10 
loose_thread_pool_oversubscribe = 10 
loose_max_statement_time = 120000 
loose_tokudb_commit_sync = ON 
binlog_stmt_cache_size = 32768 
net_retry_count = 10 
binlog_checksum = CRC32 
low_priority_updates = 0 
loose_tokudb_support_xa = ON 
loose_rds_slave_minor_log = ON 
autocommit = 1 
loose_rds_force_archive_to_tokudb = ON 
loose_rds_set_connection_id_enabled = ON 
key_cache_age_threshold = 300 
innodb_concurrency_tickets = 5000 
loose_innodb_rds_log_checksum_algorithm = INNODB 
table_definition_cache = 512 
loose_rds_binlog_group_commit_sync_delay = 0 
loose_rds_force_myisam_to_innodb = ON 
loose_rds_check_core_file_enabled = ON 
loose_tokudb_rpl_lookup_rows = OFF 
innodb_use_native_aio = 0 
net_write_timeout = 60

loose_rds_threads_running_high_watermark = 50000 
innodb_table_locks = ON 
query_alloc_block_size = 8192 
loose_tokudb_fs_reserve_percent = 5 
max_prepared_stmt_count = 16382 
loose_rds_enable_skip_counter = ON 
innodb_thread_sleep_delay = 10000 
net_read_timeout = 30 
loose_innodb_rds_min_concurrency_tickets = 50 
loose_rds_ic_reduce_hint_enable = OFF 
max_write_lock_count = 102400 
innodb_old_blocks_time = 1000 
innodb_stats_method = nullsequal 
loose_rds_deny_drop_db_contain_foreign_key = ON 
max_length_for_sort_data = 1024 
query_prealloc_size = 8192 
innodb_large_prefix = OFF 
delayed_insert_limit = 100 
group_concat_max_len = 1024 
innodb_disable_sort_file_cache = ON 
innodb_ft_min_token_size = 3 
loose_rds_enable_log_global_var_update = ON 
loose_opt_rds_last_error_gtid = ON 
loose_skip_symbolic_links = ON 
key_cache_block_size = 1024 
loose_tokudb_directio = OFF 
slow_launch_time = 2 
loose_tokudb_fsync_log_period = 0 
loose_thread_handling = "one-thread-per-connection" 
loose_rds_allow_unsafe_stmt_with_gtid = ON 
innodb_online_alter_log_max_size = 134217728 
innodb_open_files = 300 
eq_range_index_dive_limit = 10 
loose_innodb_adaptive_hash_index_parts = 8 
div_precision_increment = 4 
binlog_row_image = full 
loose_tokudb_row_format = tokudbzlib 
innodb_strict_mode = OFF 
read_only = 1 
delayed_queue_size = 1000 
default_week_format = 0 
loose_opt_rds_enable_restrict_non_super_user = ON 
loose_rds_expand_fast_index_creation = ON 
log_queries_not_using_indexes = OFF 
innodb_read_ahead_threshold = 56 
loose_rds_audit_row_limit = 100000 
loose_slave_parallel_workers = 0 
loose_rds_disable_explicit_trans = ON

default_time_zone = SYSTEM 
loose_rds_slave_read_no_lock = ON 
loose_rds_restrict_stmt_for_mscheck = ON 
sqlmode = '' 
loose_rds_enable_shield_var = ON 
slave_exec_mode = strict 
loose_opt_rds_audit_log_enabled = 1 
query_cache_size = 0 
innodb_adaptive_hash_index = ON 
performanceschema = OFF 
innodb_purge_batch_size = 300 
loose_rds_file_operation_local_only = ON 
loose_innodb_rds_adaptive_tickets_algo = ON 
loose_innodb_rds_autoinc_persistent = ON 
loose_rpl_semi_sync_slave_trace_level = 1 
loose_rpl_semi_sync_master_timeout = 1000 
loose_rpl_semi_sync_master_trace_level = 1 
loose_rpl_semi_sync_slave_enabled = ON 
loose_rpl_semi_sync_master_enabled = ON 
loose_rpl_semi_sync_master_wait_no_slave = ON
[mysqldump] quick 
max_allowed_packet = 64M

[mysql] no-auto-rehash 
prompt = "\u@\h : \d \R:\m:\s> "

[myisamchk] keybuffer = 512M 
sort_buffer_size = 512M 
readbuffer = 8M 
write_buffer = 8M

[mysqlhotcopy] interactive-timeout

[mysqld_safe] user = mysql 
basedir = /usr/local/alisql

[mysqlinstalldb] basedir = /usr/local/alisql
```

# 制作rpm
```sh
yum -y install rpm* rpm-build rpmdev* tree 
yum -y install gcc gcc-c++ ncurses-devel perl cmake bison
rpmdev-setuptree
如果是root用户，则路径是/root/rpmbuild
tree rpmbuild/
从github上下载的名字是master，解压后将文件夹改名为alisql-5.6.3，然后打包压缩
tar zcvf alisql-5.6.3.tar.gz alisql-5.6.3/
将alisql-5.6.3.tar.gz放到SOURCE目录下
在rpmbuild/SPECS目录下执行rpmdev-newspec -o alisql.spec，会在当前目录下生成名为alisql.spec的模板文件，修改内容见下一段
在rpmbuild/SPECS目录下执行打包编译
rpmbuild -bb alisql.spec
```
```sh
Name:           alisql
Version:        5.6.3
Release:        1%{?dist}
Summary:        AliSQL

Group:          Applications/Databases
License:        GPL
URL:            https://github.com/alibaba/AliSQL
Source0:        %{name}-%{version}.tar.gz
BuildRequires:  gcc gcc-c++
Requires:       ncurses-devel bison perl

%define MYSQL_USER mysql
%define MYSQL_GROUP mysql


%description    
The %{name}-devel package contains libraries and header files for
developing applications that use %{name}.


%prep
%setup -q
useradd mysql
mkdir -p /usr/local/mysql
mkdir -p /data/mysqldb

%build
cmake -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DMYSQL_UNIX_ADDR=/tmp/mysql.sock -DDEFAULT_CHARSET=utf8   -DDEFAULT_COLLATION=utf8_general_ci -DWITH_INNOBASE_STORAGE_ENGINE=1 -DWITH_ARCHIVE_STORAGE_ENGINE=1 -DWITH_BLACKHOLE_STORAGE_ENGINE=1 -DMYSQL_DATADIR=/data/mysqldb -DMYSQL_TCP_PORT=3306 -DENABLE_DOWNLOADS=1

make %{?_smp_mflags}


%install
rm -rf $RPM_BUILD_ROOT
make install DESTDIR=$RPM_BUILD_ROOT
find $RPM_BUILD_ROOT -name '*.la' -exec rm -f {} ';'

%pre
id mysql  &>/dev/null||useradd -m -s /bin/bash mysql &>/dev/null
mkdir -p /data/mysqldb
chown -R mysql: /data/mysqldb

%clean
rm -rf $RPM_BUILD_ROOT


%post 
/usr/local/mysql/scripts/mysql_install_db  --basedir=/usr/local/mysql --user=mysql  --datadir=/data/mysqldb &>/dev/null
cp -f /usr/local/mysql/support-files/my-default.cnf /etc/my.cnf 
sed -i 's/^# basedir.*/basedir=\/usr\/local\/mysql/g' /etc/my.cnf
sed -i 's/^# datadir.*/datadir=\/data\/mysqldb/g' /etc/my.cnf
sed -i 's/^# socket.*/socket= \/tmp\/mysql.sock/g' /etc/my.cnf
cp -f /usr/local/mysql/support-files/mysql.server  /etc/init.d/mysqld
echo export PATH=/usr/local/mysql/bin:/usr/local/mysql/lib:$PATH >> /etc/profile
source /etc/profile
chkconfig --add mysqld &>/dev/null
chkconfig mysqld on &>/dev/null


%preun
chkconfig --del mysqld &>/dev/null
rm -rf /etc/init.d/mysqld &>/dev/null

%postun 
userdel -r mysql &>/dev/null
rm -fr /data/mysqldb &>/dev/null
rm -fr /usr/local/mysql &>/dev/null

%files
%defattr(-,mysql,mysql,-)
/usr/local/mysql/bin
/usr/local/mysql/data
/usr/local/mysql/include
/usr/local/mysql/lib
/usr/local/mysql/scripts
/usr/local/mysql/share
/usr/local/mysql/support-files
/usr/local/mysql/README
/usr/local/mysql/docs
/usr/local/mysql/man
%exclude /usr/local/mysql/COPYING
%exclude /usr/local/mysql/mysql-test 
%exclude /usr/local/mysql/sql-bench


%changelog
```
