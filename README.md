 private static decimal GetDeciamlSafe(DbDataReader reader, string columnName)
        {
            var val = reader[columnName];
            return val != DBNull.Value ? Convert.ToDecimal(val) : 0m;
        }
