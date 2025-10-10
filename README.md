 private static T GetValueOrDefault<T>(IDataRecord reader, string columnName, T defaultValue = default!)
 {
     try
     {
         int ordinal = reader.GetOrdinal(columnName);
         return reader.IsDBNull(ordinal) ? defaultValue : (T)Convert.ChangeType(reader.GetValue(ordinal), typeof(T));
     }
     catch (IndexOutOfRangeException)
     {
         return defaultValue;
     }
     catch (InvalidCastException)
     {
         return defaultValue;
     }
 }
 Change type of parameter 'reader' from 'System.Data.IDataRecord' to 'System.Data.Common.DbDataReader' for improved performance
