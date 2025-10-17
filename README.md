  #region HelperMethods
        private static string? GetStringValue(DbDataReader reader,string columnName)
        {
            return reader[columnName] != DBNull.Value ? reader[columnName!].ToString() : "0";
        }
        private static decimal? GetDecimalValue(DbDataReader reader, string columnName)
        {
            return reader[columnName] != DBNull.Value ? Convert.ToDecimal(reader[columnName!]) : 0;
        }
        private static Int64? GetUnixTime(DbDataReader reader, string columnName)
        {
Uncovered code
            return reader[columnName] != DBNull.Value ? new DateTimeOffset(DateTime.ParseExact(reader[columnName]!.ToString()!, "yyyyMMdd", CultureInfo.InvariantCulture), TimeSpan.Zero)
                        .ToUnixTimeMilliseconds() : 0;
        }
        #endregion
