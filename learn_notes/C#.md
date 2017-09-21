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