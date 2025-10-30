private static decimal GetDecimalValue(dynamic reader, string column)
        {
            return reader[column] != DBNull.Value ? Convert.ToDecimal(reader[column]) : 0;
        }
