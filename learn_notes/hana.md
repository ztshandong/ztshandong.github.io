# 存储过程语法
```
alter  procedure "SYSTEM"."TEST_INSERT"
(in I integer,in J integer)
as begin
declare X integer := I;
while X < :J do
X := :X + 1;
insert into SYSTEM.TEST3("ID","NAME","COL1","COL2","COL3","COL4","COL5","COL6","COL7","COL8","COL21","COL22","COL23","COL24","COL25","COL26","COL51","COL52","COL53","COL54") values (X, 'A', 101 , 102, 103, 104, 105, 106, 107, 108, 1.11, 1.12, 1.13, 1.14, 1.15, 1.16,'a','b','c','d');

if MOD (X, 100)=0
then
commit;
end if;

end while;
end;

```
# 主从复制(从库无法连接)
```
hdbsql client
\c -n 192.168.1.212 -i 90 -u SYSTEM -d SYSTEMDB 
BACKUP DATA FOR FULL SYSTEM USING FILE ('backup')
将primary两个文件拷贝到secondary
/usr/sap/HXE/SYS/global/security/rsecssfs/data/SSFS_HXE.DAT
/usr/sap/HXE/SYS/global/security/rsecssfs/key/SSFS_HXE.KEY

用eclipse设置主从
master -> studio enable system replication
slave stop system -> studio register secondary system

从库启用
/usr/sap/HXE/HDB90/exe/hdbnsutil -sr_takeover
```

# 链接SQLServer
```
yast2好像有，暂时未测试

sudo zypper install libopenssl0_9_8
tar -xvf msodbcsql-SUSELinux-11.0.2260.0.tar.gz
cd msodbcsql-11.0.2260.0/
sudo ./build_dm.sh --download-url=file:///root/unixODBC-2.3.0.tar.gz
然后控制台会有个提示，Run the command
cd /root/msodbcsql-11.0.2260.0/
./install.sh verify
./install.sh install
一直回车按倒99%的时候慢一点，不要过头，然后输入YES
cp /opt/microsoft/msodbcsql/lib64/libmsodbcsql-11.0.so.2260.0 /usr/sap/HXE/HDB90/exe

ldd /opt/microsoft/msodbcsql/lib64/libmsodbcsql-11.0.so.2260.0
没有not found
使用studio映射为virtual table 就可以了

sudo vi /etc/odbc.ini

[MSSQLTESTHANA2]    --名字后面要用
Driver= /opt/microsoft/msodbcsql/lib64/libmsodbcsql-11.0.so.2260.0
Server=192.168.1.250,1433
Database=Test4HANA

eclipse
Provisioning-RemoteSources右键-NewRemoteSource
AdapterName选MSSQL，SourceName自己填例如XXX
DataSourceName填配置文件中的名字，此处为MSSQLTESTHANA2
DMLMode选readwrite
填上数据库用户名密码，然后保存（有个绿色的小箭头），保存后测试
在RemoteSource下可以看到新添加的数据源（此处为XXX）
选择一个表右键-AddAsVirtualTable，填写表名，Schema选TEST4HANA

```
