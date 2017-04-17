# CentOS7-SQLServer
```sh
yum -y install wget
wget https://packages.microsoft.com/config/rhel/7/mssql-server.repo
wget https://packages.microsoft.com/config/rhel/7/prod.repo
mv mssql-server.repo /etc/yum.repos.d/mssql.repo
mv prod.repo /etc/yum.repos.d/prod.repo

yum install -y mssql-server

/opt/mssql/bin/mssql-conf setup
/opt/mssql/bin/sqlservr -setup 

systemctl start mssql-server
systemctl enable mssql-server

yum install -y mssql-tools

firewall-cmd --permanent --zone=public --add-port=1433/tcp
echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bashrc
source ~/.bashrc
sqlcmd -H 127.0.0.1 -U sa

```
- 链接服务器229Err
- GRANT EXECUTE ON SYS.XP_PROP_OLEDB_PROVIDER TO username;

1. Sql Server插入数据并返回自增ID：@@IDENTITY，SCOPE_IDENTITY和IDENT_CURRENT的区别：@@IDENTITY是取全局操作的最后一步操作所产生的自增域的值的。如果在写存储过程时，向表中新增了一条数据，需要返回该IDENTITY值，则使用 SCOPE_IDENTITY()，因为它和当前会话相关联，不会返回一个你不需要的值。如果你仅仅想得到某个IDENTITY列的最后一个值，则使用 IDENT_CURRENT()最方便。[原文链接](http://www.bbsmax.com/A/q4zVVyylzK/)
2. 117.18.232.200 是redgate的认证IP，你懂的