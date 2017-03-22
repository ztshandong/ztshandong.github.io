# [查看最新版本，目前9.6](https://yum.postgresql.org/)
- yum -y install https://download.postgresql.org/pub/repos/yum/9.6/redhat/rhel-7-x86_64/pgdg-centos96-9.6-3.noarch.rpm
- yum -y install postgresql96-server postgresql96-contrib
- 初始化数据库
- /usr/pgsql-9.6/bin/postgresql96-setup initdb
- systemctl start postgresql-9.6.service
- systemctl enable postgresql-9.6.service
# 修改密码
```sh
PostgreSQL 安装完成后，会建立一下‘postgres’用户，用于执行PostgreSQL，
数据库中也会建立一个'postgres'用户，默认密码为自动生成，需要在系统中改一下。
passwd postgres  为默认创建的系统用户设置个密码
su  postgres  切换用户，执行后提示符会变为 '-bash-4.2$'
psql -U postgres 登录数据库，执行后提示符变为 'postgres=#'
ALTER USER postgres WITH PASSWORD '123456'  设置postgres用户密码
\q  退出数据库
```
# 开启远程访问
- vi /var/lib/pgsql/9.6/data/postgresql.conf
- 修改#listen_addresses = 'localhost'  为  listen_addresses='*'
- ‘*’也可以改为任何你想开放的服务器IP
- vi /var/lib/pgsql/9.6/data/pg_hba.conf
- # IPv4 local connections:
- host  all    all    192.168.1.0/24      trust
- host  all    all    0.0.0.0    md5
# 防火墙
- firewall-cmd --permanent --zone=public --add-service=postgresql
