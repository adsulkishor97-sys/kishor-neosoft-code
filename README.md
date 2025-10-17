  #region HelperMethods
        private static string? GetStringValue(DbDataReader reader, string columnName)
        {
            return reader[columnName] != DBNull.Value ? reader[columnName!].ToString() : "0";
        }
        private static Int64? GetIntegerValue(DbDataReader reader, string columnName)
        {
            return reader[columnName] == DBNull.Value || reader[columnName].ToString() == "NaN" || reader[columnName].ToString() == "Infinity" ? 0 : Convert.ToInt64(reader[columnName]);
        }
         private static decimal GetDeciamlSafe(DbDataReader reader, string columnName)
        {
Uncovered code
            var val = reader[columnName];
            return val != DBNull.Value ? Convert.ToDecimal(val) : 0m;
        }
