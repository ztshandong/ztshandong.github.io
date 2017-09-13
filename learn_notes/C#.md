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