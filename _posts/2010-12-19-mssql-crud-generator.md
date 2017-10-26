---
layout: post
title: mssql crud generator

tags: [.net, c#, mssql]
---

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Data;
using System.Data.SqlClient;
using System.Collections;

namespace dsql
{
    class Program
    {
        static void Main(string[] args)
        {
            string connectionString = "Data Source=BETASRV;Initial Catalog=RabotaUA2;User ID=sa;Password=1;Persist Security Info=true;";
            string TableName = "be_Posts";

            Console.WriteLine(SCRUDQG.Generator.Create(connectionString, TableName));
            Console.WriteLine("-----------------------------------------------------------");
            Console.WriteLine(SCRUDQG.Generator.Read(connectionString, TableName));
            Console.WriteLine("-----------------------------------------------------------");
            Console.WriteLine(SCRUDQG.Generator.Read(connectionString, TableName, true));
            Console.WriteLine("-----------------------------------------------------------");
            Console.WriteLine(SCRUDQG.Generator.Update(connectionString, TableName));
            Console.WriteLine("-----------------------------------------------------------");
            Console.WriteLine(SCRUDQG.Generator.Delete(connectionString, TableName));

            Console.ReadKey();
        }
    }
}

namespace SCRUDQG
{
    public class Generator
    {
        public static string Create(string connectionString, string TableName)
        {
            StringBuilder tpl = new StringBuilder();
            tpl.AppendLine("SET NOCOUNT ON;");
            tpl.AppendLine("INSERT INTO {0} (");
            tpl.AppendLine("{1}");
            tpl.AppendLine(") VALUES (");
            tpl.AppendLine("{2}");
            tpl.AppendLine(");");
            tpl.AppendLine("SELECT @@IDENTITY AS LAST_ID;");
            tpl.AppendLine("SET NOCOUNT OFF;");

            string identityColumnName = Helper.getTableIdentity(connectionString, TableName);
            List<string> columns = new List<string>(Helper.getTableColumns(connectionString, TableName).Replace(identityColumnName,"").Replace(",,",",").Trim(',').Split(','));

            StringBuilder sbColumns = new StringBuilder();
            StringBuilder sbValues = new StringBuilder();

            foreach (string col in columns)
            {
                sbColumns.AppendFormat("\t{0},{1}",col, Environment.NewLine);
                sbValues.AppendFormat("\t@{0},{1}", col, Environment.NewLine);
            }

            return string.Format(tpl.ToString(), TableName, sbColumns.ToString().Substring(0, sbColumns.ToString().LastIndexOf(',')), sbValues.ToString().Substring(0, sbValues.ToString().LastIndexOf(',')));
        }

        public static string Read(string connectionString, string TableName)
        {
            return Generator.Read(connectionString, TableName, false);
        }

        public static string Read(string connectionString, string TableName, bool ById)
        {
            string result = string.Format("SELECT * FROM {0}", TableName);
            if (ById) result = string.Format("{0} WHERE {1} = @{1}", result, Helper.getTableIdentity(connectionString, TableName));
            return result;
        }

        public static string Update(string connectionString, string TableName)
        {
            StringBuilder tpl = new StringBuilder();
            tpl.AppendLine("UPDATE {0} SET ");
            tpl.AppendLine("{1}");
            tpl.AppendLine("WHERE {2} = @{2}");

            string identityColumnName = Helper.getTableIdentity(connectionString, TableName);
            List<string> columns = new List<string>(Helper.getTableColumns(connectionString, TableName).Replace(identityColumnName, "").Replace(",,", ",").Trim(',').Split(','));

            string max = Helper.maxColumnNameWidth(columns);
            StringBuilder sbColumns = new StringBuilder();
            foreach (string col in columns)
            {
                sbColumns.AppendFormat("\t{0, -" + max + "} = @{0},{1}", col, Environment.NewLine);
            }

            return string.Format(tpl.ToString(), TableName, sbColumns.ToString().Substring(0, sbColumns.ToString().LastIndexOf(',')), Helper.getTableIdentity(connectionString, TableName));
        }

        public static string Delete(string connectionString, string TableName)
        {
            return string.Format("DELETE FROM {0} WHERE {1} = @{1}", TableName, Helper.getTableIdentity(connectionString, TableName));
        }
    }

    public class Helper
    {
        static Helper() { }

        public static object ExecuteScalar(string connectionString, string sqlQuery)
        {
            return ExecuteScalar(connectionString, sqlQuery, new List<SqlParameter>());
        }

        public static object ExecuteScalar(string connectionString, string sqlQuery, List<SqlParameter> sqlParameters)
        {
            object result = null;
            try
            {
                using (SqlConnection cn = new SqlConnection(connectionString))
                {
                    cn.Open();
                    SqlCommand cmd = new SqlCommand(sqlQuery, cn);

                    foreach (SqlParameter param in sqlParameters)
                    {
                        cmd.Parameters.Add(param);
                    }

                    object oResult = cmd.ExecuteScalar();
                    if (oResult != null)
                    {
                        result = oResult;
                    }
                    cmd.Dispose();
                    cn.Close();
                }
            }
            catch (SqlException sqlex)
            {
                throw sqlex;
            }
            catch (Exception ex)
            {
                throw ex;
            }
            return result;
        }

        public static bool isTableExists(string connectionString, string tableName)
        {
            int total = (int)Helper.ExecuteScalar(connectionString, "SELECT COUNT(*) AS total FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_NAME = @TableName", new List<SqlParameter>() { new SqlParameter("TableName", tableName) });
            return (total == 1);
        }

        public static string getTableIdentity(string connectionString, string tableName)
        {
            if (!Helper.isTableExists(connectionString, tableName)) throw new GeneratorException(string.Format("There is no table \"{0}\"", tableName));

            object oResult = Helper.ExecuteScalar(connectionString, "SELECT COLUMN_NAME FROM INFORMATION_SCHEMA.COLUMNS WHERE (COLUMNPROPERTY(OBJECT_ID(TABLE_NAME), COLUMN_NAME, 'IsRowGuidCol') = 1 OR COLUMNPROPERTY(OBJECT_ID(TABLE_NAME), COLUMN_NAME, 'IsIdentity') = 1) AND TABLE_NAME = @TableName", new List<SqlParameter>() { new SqlParameter("TableName", tableName) });

            if (oResult == null) throw new GeneratorException(string.Format("There is no identity field in \"{0}\" table", tableName));

            return oResult.ToString();
        }

        public static string getTableColumns(string connectionString, string tableName)
        {
            if (!Helper.isTableExists(connectionString, tableName)) throw new GeneratorException(string.Format("There is no table \"{0}\"", tableName));

            object oResult = Helper.ExecuteScalar(connectionString, "SELECT LEFT(column_names , LEN(column_names )-1) AS column_names FROM INFORMATION_SCHEMA.COLUMNS AS extern CROSS APPLY ( SELECT COLUMN_NAME + ',' FROM INFORMATION_SCHEMA.COLUMNS AS intern WHERE extern.TABLE_NAME = intern.TABLE_NAME FOR XML PATH('') ) pre_trimmed (column_names) WHERE TABLE_NAME = @TableName GROUP BY column_names;", new List<SqlParameter>() { new SqlParameter("TableName", tableName) });

            if (oResult == null) throw new GeneratorException(string.Format("There is no columns in \"{0}\" table", tableName));

            return oResult.ToString();
        }

        public static string maxColumnNameWidth(List<string> columns)
        {
            int max = 0;
            foreach (string col in columns)
            {
                if (col.Length > max) max = col.Length;
            }
            return max.ToString();
        }
    }

    public class GeneratorException : System.Exception
    {
        public GeneratorException() { }
        public GeneratorException(string message) : base(message) { }
        public GeneratorException(string message, Exception innerException) : base(message, innerException) { }
    }
}
```
