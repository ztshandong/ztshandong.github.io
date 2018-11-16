# 安装
```sh
boot bootmgr 和sources复制到C盘根目录
c:\boot\bootsect.exe /nt60 c:
```

# 2016故障转移集群
```
安装故障转移服务后服务管理器有BUG，无法显示磁盘、卷、iSCSI等信息
所以存储服务器独立出来就行
所有电脑用户名密码一样
创建DNSserver或者修改计算机名与DNS后缀，hosts文件
关防火墙关防火墙关防火墙


一、两块空白硬盘
二、新建存储池，相当于把两块硬盘合成一个
三、新建虚拟磁盘，相当于把磁盘分区，10G的当仲裁盘，大的当数据盘，然后新建卷
磁盘分以下几类
仲裁盘-要作为集群仲裁磁盘
集群磁盘-只有活动节点可用
集群共享磁盘-所有节点都可以显示
独立磁盘-每个节点都连接一个，最好挂载到文件夹下，因为故障转移集群所有节点文件位置必须一样，但是不能用同一个文件
共享盘-所有人都可以用的


四、新建iSCSI磁盘，为了让其他服务器连接，然后貌似要重启，否则3620端口好像打不开
五、其他电脑-控制面板-iSCSI发起程序，目标填写ip，卷和设备自动配置
创建故障转移集群，添加两台电脑
修改群集ip
两台电脑计算机管理，磁盘联机，初始化MBR，转为基本磁盘，不要创建卷
六、故障转移集群管理器-存储-磁盘-添加磁盘
七、计算机管理-磁盘管理-新建简单卷-注意不要格式化
八、我的电脑-格式化
仲裁盘不能群集磁盘共享
九、故障转移集群管理器-群集磁盘共享，将数据磁盘共享，所有电脑都可以在C:\ClusterStorage下看到，但是群集共享卷很不稳定，很卡，否则只有一台电脑可以看到当前磁盘
十、故障转移集群-磁盘仲裁



iscsi服务器重启后iscsi虚拟磁盘会显示为空，错误信息为Microsoft iSCSI Target Server服务无法加载vhdx文件，此时重启Microsoft iSCSI Target Server服务，因为系统盘为固态硬盘，iscsi磁盘为机械硬盘，感觉是服务启动时机械硬盘还未准备好，iscsi磁盘正常显示后才可以启动其他服务器

可使用hype-v中虚拟机与真机共同集群，存储服务器安装个虚拟机

安装SQL代理与引擎用admin用户，不要安装数据库补丁
两台电脑配置管理器SQL服务启用alwayson
创建证书
--共享文件夹路径： ---\\StorageServer.bugong.com\AllShareDisk1

--节点Clus2-Server1上执行：创建主密钥/证书/端点，备份证书。 
USE master; 
GO 

CREATE MASTER KEY ENCRYPTION BY PASSWORD = '123456'; ----密码
GO 

CREATE CERTIFICATE Cert_Clus2Server1
WITH SUBJECT = 'Cert_Clus2Server1', 
START_DATE = '2017-12-01',EXPIRY_DATE = '2099-12-31'; 
GO 

BACKUP CERTIFICATE Cert_Clus2Server1
TO FILE = '\\StorageServer.bugong.com\AllShareDisk1\Cert_Clus2Server1.cer'; 
GO 

CREATE ENDPOINT [SQLAG_Endpoint] 
AUTHORIZATION [Clus2-Server1\administrator] 
STATE=STARTED 
AS TCP (LISTENER_PORT = 5022, LISTENER_IP = ALL) 
FOR DATA_MIRRORING 
(ROLE = ALL,AUTHENTICATION = CERTIFICATE Cert_Clus2Server1, ENCRYPTION = REQUIRED ALGORITHM AES)
GO 



--节点Clus2-Server2上执行：创建主密钥/证书，备份证书。 
USE master; 
GO 
CREATE MASTER KEY ENCRYPTION BY PASSWORD = '123456'; 
GO 

CREATE CERTIFICATE Cert_Clus2Server2
WITH SUBJECT = 'Cert_Clus2Server2', 
START_DATE = '2017-12-01',EXPIRY_DATE = '2099-12-31'; 
GO 

BACKUP CERTIFICATE Cert_Clus2Server2
TO FILE = '\\StorageServer.bugong.com\AllShareDisk1\Cert_Clus2Server2.cer'; 
GO 

CREATE ENDPOINT [SQLAG_Endpoint] 
AUTHORIZATION [Clus2-Server2\administrator] 
STATE=STARTED 
AS TCP (LISTENER_PORT = 5022, LISTENER_IP = ALL) 
FOR DATA_MIRRORING 
(ROLE = ALL,AUTHENTICATION = CERTIFICATE Cert_Clus2Server2, ENCRYPTION = REQUIRED ALGORITHM AES)
GO 




--节点Clus2-Server1上执行：创建节点Clus2-Server2的证书 
USE master; 
GO 
CREATE CERTIFICATE Cert_Clus2Server2
FROM FILE = '\\StorageServer.bugong.com\AllShareDisk1\Cert_Clus2Server2.cer'; 
GO 


--节点Clus2-Server2上执行：创建节点Clus2-Server1的证书 
USE master; 
GO 
CREATE CERTIFICATE Cert_Clus2Server1
FROM FILE = '\\StorageServer.bugong.com\AllShareDisk1\Cert_Clus2Server1.cer'; 
GO 

创建数据库-备份
新建可用性组向导
数据同步-自动种子
添加侦听器-只读路由
故障转移集群-角色-属性设置超时次数为10




ALTER AVAILABILITY GROUP [SQLAlwaysOnGroup]
 MODIFY REPLICA ON  
N'Clus1-Server1' WITH   
(SECONDARY_ROLE (ALLOW_CONNECTIONS = READ_ONLY));  
ALTER AVAILABILITY GROUP [SQLAlwaysOnGroup]
 MODIFY REPLICA ON  
N'Clus1-Server1' WITH   
(SECONDARY_ROLE (READ_ONLY_ROUTING_URL = N'TCP://Clus1-Server1.bugong.com:1433'));  

ALTER AVAILABILITY GROUP [SQLAlwaysOnGroup]
 MODIFY REPLICA ON  
N'SQL-MASTER' WITH   
(SECONDARY_ROLE (ALLOW_CONNECTIONS = READ_ONLY));  
ALTER AVAILABILITY GROUP [SQLAlwaysOnGroup]
 MODIFY REPLICA ON  
N'SQL-MASTER' WITH   
(SECONDARY_ROLE (READ_ONLY_ROUTING_URL = N'TCP://SQL-Master.bugong.com:1433'));  

ALTER AVAILABILITY GROUP [SQLAlwaysOnGroup]
 MODIFY REPLICA ON  
N'SQL-SLAVE' WITH   
(SECONDARY_ROLE (ALLOW_CONNECTIONS = READ_ONLY));  
ALTER AVAILABILITY GROUP [SQLAlwaysOnGroup]
 MODIFY REPLICA ON  
N'SQL-SLAVE' WITH   
(SECONDARY_ROLE (READ_ONLY_ROUTING_URL = N'TCP://SQL-Slave.bugong.com:1433'));  


多个读副本负载只能命令行


ALTER AVAILABILITY GROUP [SQLAlwaysOnGroup]
MODIFY REPLICA ON
N'SQL-MASTER' WITH
(PRIMARY_ROLE (READ_ONLY_ROUTING_LIST=(('SQL-SLAVE','Clus1-Server1'),'SQL-MASTER')));

ALTER AVAILABILITY GROUP [SQLAlwaysOnGroup]
MODIFY REPLICA ON
N'Clus1-Server1' WITH
(PRIMARY_ROLE (READ_ONLY_ROUTING_LIST=(('SQL-MASTER','SQL-SLAVE'),'Clus1-Server1')));

ALTER AVAILABILITY GROUP [SQLAlwaysOnGroup]
MODIFY REPLICA ON
N'SQL-SLAVE' WITH
(PRIMARY_ROLE (READ_ONLY_ROUTING_LIST=(('Clus1-Server1','SQL-MASTER'),'SQL-SLAVE')));


```