- 链接服务器229Err
- GRANT EXECUTE ON SYS.XP_PROP_OLEDB_PROVIDER TO username;

# CentOS7-SQLServer，你不会真的在CentOS上还要装SQLServer吧，看看就行
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
      FROM #tb_ACC_FeeApplyDetail WHERE DepartmentCode=@DepartmentCode AND OrgCode=@OrgCode
      ) x
```
# 多行转一行
```sql
SELECT TOP 100 A.YDNO,A.DocDate, ZZANDDT=
STUFF(
(SELECT ','+B.DocNo FROM dbo.tb_TMS_CKs AS B
WHERE B.YDNO=A.YDNO FOR XML PATH (''))
,1,1,'')
FROM dbo.tb_TMS_YD A WHERE 1=1
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
1. Sql Server插入数据并返回自增ID：@@IDENTITY，SCOPE_IDENTITY和IDENT_CURRENT的区别：@@IDENTITY是取全局操作的最后一步操作所产生的自增域的值的。如果在写存储过程时，向表中新增了一条数据，需要返回该IDENTITY值，则使用 SCOPE_IDENTITY()，因为它和当前会话相关联，不会返回一个你不需要的值。如果你仅仅想得到某个IDENTITY列的最后一个值，则使用 IDENT_CURRENT()最方便。[原文链接](http://www.bbsmax.com/A/q4zVVyylzK/)
2. 117.18.232.200 是redgate的认证IP，你懂的