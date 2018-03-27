# SQL跨库同步存储过程
```sh
static string Current_Constr = "xxxx";
static string TargerConstr = "xxx";
public static void SyncUp_DataBase_Schema()
{
    string type = "('v','fn','tn','p')";
    string sqlCmd = "select object_id,definition,name,xtype FROM sys.sql_modules  inner join sysobjects  on sys.sql_modules.object_id = sysobjects.id where  xtype in {0}";
    string sql = string.Format(sqlCmd, type);
   var dicts  = GetSqlString(sql);
    ExecuteSql(dicts);
}
 
private static void ExecuteSql(Dictionary<string, string> dicts)
{
    using (SqlConnection conn = new SqlConnection(TargerConstr))
    {
     conn.Open();
     SqlTransaction tran = conn.BeginTransaction();
     SqlCommand cmd = new SqlCommand();
     cmd.Transaction = tran;
     cmd.Connection = conn;
     try
     {
         foreach (var item in dicts)
         {
             cmd.CommandText = item.Key;
             cmd.ExecuteNonQuery();
             cmd.CommandText = item.Value;
             cmd.ExecuteNonQuery();
         }
     }
     catch
     {
         tran.Rollback();
     }
     finally {
         tran.Commit();
         tran.Dispose();
     }
    }
}
 
private static Dictionary<string, string> GetSqlString(string sql)
{
    Dictionary<string, string> dicts = new Dictionary<string, string>();
    DataTable dt = new DataTable();
    
    using (SqlDataAdapter da = new SqlDataAdapter(sql, Current_Constr))
    {
        da.Fill(dt);
    }
    foreach (DataRow dr in dt.Rows)
    {
        string type = "function";
        string xtype = dr[3].ToString().ToLower().Trim();
        string str = "if object_id(N'{0}') is not null drop {1} {0} ";
       
        if (xtype == "v")
        {
            type = "view";
        }
        else if(xtype=="p"){
            type = "proc";
        }
        string dropSql = string.Format(str, dr[2].ToString(), type);
        dicts.Add(dropSql,dr[1].ToString());
    }
    return dicts;
}
```

# 存储过程
```c#
   static string MessageDBConnStr { get { return @"Data Source=192.168.1.2:1521/xe;user Id=oracle;Password=oracle"; } }
        //static string MessageDBConnStr { get { return @"server=192.168.1.2;user id=mysql;password=mysql;database=dbname;port=3307"; } }
        //static string MessageDBConnStr { get { return @"Data Source=192.168.1.2;Initial Catalog=dbname;Persist Security Info=True;User ID=sqlserver;Password=sqlserver;MultipleActiveResultSets=true"; } }

        //return (IDbConnection)new System.Data.SqlClient.SqlConnection(MessageDBConnStr);
        //return (IDbConnection)new MySql.Data.MySqlClient.MySqlConnection(MessageDBConnStr);

            return (IDbConnection)new Oracle.ManagedDataAccess.Client.OracleConnection(MessageDBConnStr);


            List<Class1> list = new List<Class1>();
            IDbCommand cmd = Connection.CreateCommand();
            cmd.CommandText = "usp_name";
            cmd.CommandType = CommandType.StoredProcedure;

            IDbDataParameter dbp1 = cmd.CreateParameter();
            dbp1.ParameterName = "@param1";
            dbp1.Value = userInfo.ID;
            dbp1.DbType = DbType.Int32;
            cmd.Parameters.Add(dbp1);

            if (cmd is Oracle.ManagedDataAccess.Client.OracleCommand)
            {
                Oracle.ManagedDataAccess.Client.OracleParameter dbp2 = (Oracle.ManagedDataAccess.Client.OracleParameter)cmd.CreateParameter();
                dbp2.ParameterName = "@my_cur";
                dbp2.Direction = ParameterDirection.Output;
                dbp2.OracleDbType = Oracle.ManagedDataAccess.Client.OracleDbType.RefCursor;
                cmd.Parameters.Add(dbp2);
            }
            IDataReader msgReader = cmd.ExecuteReader();
            
            
            int FieldCount = msgReader.FieldCount;
            //
            int[] FieldLocation = new int[2] { -1, -1};
            for (int i = 0; i < msgReader.FieldCount; ++i)
            {
                if (string.Compare(msgReader.GetSchemaTable().Rows[i].ItemArray[0].ToString(), "PARAM1", true) == 0)
                {
                    FieldLocation[0] = i;
                }
                else if (string.Compare(msgReader.GetSchemaTable().Rows[i].ItemArray[0].ToString(), "PARAM2", true) == 0)
                {
                    FieldLocation[1] = i;
                }
            }
            while (msgReader.Read())
            {
                Class1 usr = new Class1();

                if (FieldLocation[0] > -1)
                {
                    usr.Param1 = (string)msgReader[FieldLocation[0]];
                }
                if (FieldLocation[1] > -1)
                {
                    usr.Param2 = Convert.ToInt32(msgReader[FieldLocation[1]]);
                }
                list.add(usr);
            }
```

# DataBaseConnFactory
```c#
App.config
  <connectionStrings>
    <add name="Conn1" connectionString="Server=127.0.0.1,1234;Database=DBName;User ID=sa;Password=123456; Max Pool Size=512; Pooling=true;Connect Timeout=30" providerName="System.Data.SqlClient" />
    <add name="EncryptKey" connectionString="123456" />
  </connectionStrings>

  public class DataBaseConnFactory
    {
        private string Conn1 = System.Configuration.ConfigurationManager.ConnectionStrings[WebApiGlobal.Conn1].ConnectionString;

        private static Dictionary<ChooseDataBase, string> Conn4StringDataBase = new Dictionary<ChooseDataBase, string>();
        private static Dictionary<ChooseDataBase, string> CheckDataBase4UserKey = new Dictionary<ChooseDataBase, string>();

        private static readonly Lazy<DataBaseConnFactory> lazy = new Lazy<DataBaseConnFactory>(() => new DataBaseConnFactory());
        public static DataBaseConnFactory Instance { get { return lazy.Value; } }

        private DataBaseConnFactory()
        {
            Conn4StringDataBase.Add(ChooseDataBase.Test, Conn1);
            CheckDataBase4UserKey.Add(ChooseDataBase.Test, ChooseDataBase.Test.ToString());
        }
        public string GetConnString(ChooseDataBase chooseDataBase)
        {
            return Conn4StringDataBase[chooseDataBase];
        }
       
    }


public class ProviderFactory
    {
        private static Dictionary<DbProviderType, string> providerInvariantNames = new Dictionary<DbProviderType, string>();
        private static Dictionary<DbProviderType, DbProviderFactory> providerFactoies = new Dictionary<DbProviderType, DbProviderFactory>(20);
        private static readonly Lazy<ProviderFactory> lazy = new Lazy<ProviderFactory>(() => new ProviderFactory());
        public static ProviderFactory Instance { get { return lazy.Value; } }
        private ProviderFactory()
        {
            //加载已知的数据库访问类的程序集  
            providerInvariantNames.Add(DbProviderType.SqlServer, "System.Data.SqlClient");
            //providerInvariantNames.Add(DbProviderType.OleDb, "System.Data.OleDb");//Excel单独处理
            providerInvariantNames.Add(DbProviderType.ODBC, "System.Data.ODBC");
            providerInvariantNames.Add(DbProviderType.Oracle, "Oracle.DataAccess.Client");
            providerInvariantNames.Add(DbProviderType.MySql, "MySql.Data.MySqlClient");
            providerInvariantNames.Add(DbProviderType.SQLite, "System.Data.SQLite");
            providerInvariantNames.Add(DbProviderType.Firebird, "FirebirdSql.Data.Firebird");
            providerInvariantNames.Add(DbProviderType.PostgreSql, "Npgsql");
            providerInvariantNames.Add(DbProviderType.DB2, "IBM.Data.DB2.iSeries");
            providerInvariantNames.Add(DbProviderType.Informix, "IBM.Data.Informix");
            providerInvariantNames.Add(DbProviderType.SqlServerCe, "System.Data.SqlServerCe");
        }
        /// <summary>  
        /// 获取指定数据库类型对应的程序集名称  
        /// </summary>  
        /// <param name="providerType">数据库类型枚举</param>  
        /// <returns></returns>  
        internal static string GetProviderInvariantName(DbProviderType providerType)
        {
            return providerInvariantNames[providerType];
        }
        /// <summary>  
        /// 获取指定类型的数据库对应的DbProviderFactory  
        /// </summary>  
        /// <param name="providerType">数据库类型枚举</param>  
        /// <returns></returns>  
        public DbProviderFactory GetDbProviderFactory(DbProviderType providerType)
        {
            //如果还没有加载，则加载该DbProviderFactory  
            if (!providerFactoies.ContainsKey(providerType))
            {
                providerFactoies.Add(providerType, ImportDbProviderFactory(providerType));
            }
            return providerFactoies[providerType];
        }
        /// <summary>  
        /// 加载指定数据库类型的DbProviderFactory  
        /// </summary>  
        /// <param name="providerType">数据库类型枚举</param>  
        /// <returns></returns>  
        private DbProviderFactory ImportDbProviderFactory(DbProviderType providerType)
        {
            string providerName = providerInvariantNames[providerType];
            DbProviderFactory factory = null;
            try
            {
                //从全局程序集中查找  
                factory = DbProviderFactories.GetFactory(providerName);
            }
            catch (ArgumentException e)
            {
                factory = null;
            }
            return factory;
        }
    }


 internal class DataProvider
    {
        private static readonly Lazy<DataProvider> lazy = new Lazy<DataProvider>(() => new DataProvider());

        public static DataProvider Instance { get { return lazy.Value; } }

        Dictionary<Type, DbType> typeMap = new Dictionary<Type, DbType>();
        private DataProvider()
        { IniDbType(); }

        private void IniDbType()
        {
            typeMap[typeof(object)] = DbType.Object;
            typeMap[typeof(byte)] = DbType.Byte;
            typeMap[typeof(sbyte)] = DbType.SByte;
            typeMap[typeof(short)] = DbType.Int16;
            typeMap[typeof(ushort)] = DbType.UInt16;
            typeMap[typeof(int)] = DbType.Int32;
            typeMap[typeof(uint)] = DbType.UInt32;
            typeMap[typeof(long)] = DbType.Int64;
            typeMap[typeof(ulong)] = DbType.UInt64;
            typeMap[typeof(float)] = DbType.Single;
            typeMap[typeof(double)] = DbType.Double;
            typeMap[typeof(decimal)] = DbType.Decimal;
            typeMap[typeof(bool)] = DbType.Boolean;
            typeMap[typeof(string)] = DbType.String;
            typeMap[typeof(char)] = DbType.StringFixedLength;
            typeMap[typeof(Guid)] = DbType.Guid;
            typeMap[typeof(DateTime)] = DbType.DateTime;
            typeMap[typeof(DateTimeOffset)] = DbType.DateTimeOffset;
            typeMap[typeof(byte[])] = DbType.Binary;
            typeMap[typeof(byte?)] = DbType.Byte;
            typeMap[typeof(sbyte?)] = DbType.SByte;
            typeMap[typeof(short?)] = DbType.Int16;
            typeMap[typeof(ushort?)] = DbType.UInt16;
            typeMap[typeof(int?)] = DbType.Int32;
            typeMap[typeof(uint?)] = DbType.UInt32;
            typeMap[typeof(long?)] = DbType.Int64;
            typeMap[typeof(ulong?)] = DbType.UInt64;
            typeMap[typeof(float?)] = DbType.Single;
            typeMap[typeof(double?)] = DbType.Double;
            typeMap[typeof(decimal?)] = DbType.Decimal;
            typeMap[typeof(bool?)] = DbType.Boolean;
            typeMap[typeof(char?)] = DbType.StringFixedLength;
            typeMap[typeof(Guid?)] = DbType.Guid;
            typeMap[typeof(DateTime?)] = DbType.DateTime;
            typeMap[typeof(DateTimeOffset?)] = DbType.DateTimeOffset;
        }

        private DbConnection CreateConnection(DbProvider4DAL dbp)
        {
            string conn = DataBaseConnFactory.Instance.GetConnString(dbp.CurrentChooseDataBase);
            DbConnection DBConn = ProviderFactory.Instance.GetDbProviderFactory(dbp.CurrentDbProviderType).CreateConnection();
            DBConn.ConnectionString = conn;
            return DBConn;
        }

        internal DataSet GetDataSet(DbProvider4DAL dbp)
        {
            try
            {
                DataSet ds = new DataSet();
                if (dbp.CurrentCommand.Connection == null)
                {
                    dbp.CurrentCommand.Connection = CreateConnection(dbp);
                }
                using (dbp.CurrentCommand.Connection)
                {
                    dbp.CurrentDbDataAdapter.Fill(ds);
                }
                
                return ds;
            }
            catch (Exception ex) { throw ex; }
            finally
            {
                dbp.CurrentDbDataAdapter.Dispose();
                dbp.CurrentCommand.Connection.Close();
                dbp.CurrentCommand.Connection.Dispose();
            }
        }
        internal void AddDbParamsByCustom(DbCommand dbcomm, string ParameterName, object ParameterValue, DbType dbType)
        {
            DbParameter dbp = dbcomm.CreateParameter();
            dbp.ParameterName = ParameterName;
            dbp.Value = ParameterValue;
            dbp.DbType = dbType;
            dbcomm.Parameters.Add(dbp);
        }
        internal void AddDbParamsByPropertyInfo(DbCommand dbcomm, object parameters = null)
        {
            if (parameters != null)
            {
                Type T = parameters.GetType();
                PropertyInfo[] properties = T.GetProperties();
                foreach (PropertyInfo property in properties)
                {
                    object value = property.GetValue(parameters);

                    if ((!value.IsNullOrEmpty()) && (!dbcomm.Parameters.Contains("@" + property.Name)))
                    {
                        DbParameter dbp = dbcomm.CreateParameter();
                        dbp.ParameterName = "@" + property.Name;
                        dbp.Value = property.GetValue(parameters);
                        dbp.DbType = typeMap[property.PropertyType];
                        if (dbp.DbType == DbType.DateTime)
                            if (dbp.Value.ToSqlDateTime() <= WebApiGlobal.MinSqlDate) continue;
                        if (dbp.DbType == DbType.Int16 || dbp.DbType == DbType.Int32 || dbp.DbType == DbType.Int64
                            || dbp.DbType == DbType.Single || dbp.DbType == DbType.Double || dbp.DbType == DbType.Decimal
                            )
                            if (dbp.Value.ToDecimalEx() == 0) continue;

                        dbcomm.Parameters.Add(dbp);
                    }
                }
            }
        }

        internal void AddDbParamsByFieldInfo(DbCommand dbcomm, object parameters = null)
        {
            if (parameters != null)
            {
                Type T = parameters.GetType();
                
                FieldInfo[] fields = GetAllInfos(T);
                foreach (FieldInfo field in fields)
                {
                    object value = field.GetValue(parameters);
                    if ((!value.IsNullOrEmpty()) && (!dbcomm.Parameters.Contains("@" + field.Name)))
                    {
                        DbParameter dbp = dbcomm.CreateParameter();
                        dbp.ParameterName = "@" + field.Name;
                        dbp.Value = field.GetValue(parameters);
                        dbp.DbType = typeMap[field.FieldType];
                        dbcomm.Parameters.Add(dbp);
                    }
                }
            }
        }
        private FieldInfo[] GetAllInfos(Type T)
        {
            FieldInfo[] infos = null;
            FieldInfo[] infosSelf = T.GetFields();
            FieldInfo[] infosBase = null;

            if (T.BaseType != null)
                infosBase = GetAllInfos(T.BaseType);

            if (infosBase == null || infosBase.Length == 0)
                infos = infosSelf;
            else
            {
                int iLength = infosBase.Length;
                Array.Resize<FieldInfo>(ref infosBase, infosBase.Length + infosSelf.Length);
                infosSelf.CopyTo(infosBase, iLength);
                infos = infosBase;
            }
            return infos;
        }
    }

```

# 嵌入excel
```c#
一、下载dsoframer.ocx
二、32位放入System32，64位放入SysWOW64文件夹下
三、对应文件夹下运行regsvr32 dsoframer.ocx
四、工具箱COM组件中即可显示，勾选上之后工具箱中即可使用
五、axFramerControl1.Open(System.IO.Directory.GetCurrentDirectory() + "\\excel\\xxx.xls"
六、要手动点击左上方文件-保存，快捷键无效，应该是与内置快捷键冲突
```
