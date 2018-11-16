# DB2集群
```
win2016+db2.11.2
1.创建域，安装db2时用域用户，启用操作系统安全性选择域
第二台安装时选择连接现有分区数据库环境
2.这一步好像不能用：slave机器上db2stop force ; 删了会出问题,db2idrop db2

3.master
db2icrt db3
set db2instance=db3
设置IP_NAME的DNS
创建db2mscs.cfg
DB2_INSTANCE =DB3

CLUSTER_NAME =HV-DB2-Clus1

DB2_LOGON_USERNAME =BUGONG\db2admin

DB2_LOGON_PASSWORD =123456

GROUP_NAME=DB2Group2

IP_NAME=db2clusip3

IP_ADDRESS= 192.168.1.36

IP_SUBNET=255.255.255.0

IP_NETWORK=DB2Net1

NETNAME_NAME=db2netname3

NETNAME_VALUE=db2netvalue3

NETNAME_DEPENDENCY=db2clusip3

DISK_NAME =Disk1

INSTPROF_DISK =E:


4.master
运行db2cmd
db2mscs -f c:\db2mscs.cfg

角色设置为允许故障恢复

DB2SET DB2_FALLBACK=ON

5.db2 create database TEST3 on E:   这一步比较久

db2 list db directory on E:

db2
connect to sample/testdb2

set db2instance=DB3
db2set db2instdef=DB3
db2 catalog local node nodeLDB3 instance DB3
db2 catalog TCPIP node nodeLDB3 remote 192.168.1.31 server 50000
db2 catalog db TEST3 as TEST3 at node nodeLDB3
DB2 TERMINATE 

db2set DB2COMM=TCPIP
db2 update dbm cfg using svcename db2c_db2       DB2_db2inst1

db2stop
db2start

nuget;
ibm.data.db2
ibm.data.db.provider
string strConn = @"SERVER=192.168.1.38:50000;Database=testdb2;UID=bugong\db2admin;PWD=123456;";
string strSql = @"select * from sysibm.sysversions";

db2ilist
db2 update dbm cfg using dftdbpath E:
db2set -all
db2 get dbm cfg
db2 catalog db TESTDB2 as TESTDB2 on E:
db2 uncatalog db dbAlisName
CONNECT TO local_db_name
USER userid
USING password

下面是编目时常用命令：

1. LIST NODE DIRECTORY         --查看客户机目录节点

2. UNCATALOG NODE node_name    --删除编目节点node_name

3. TERMINATE         --刷新目录高速缓存

4. LIST DB DIRECTORY             --查看本地数据库目录

5. UNCATALOG DB db_name       --删除数据库编目db_name
```