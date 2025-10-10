private static T GetValueOrDefault<T>(DbDataReader reader, string columnName, T defaultValue = default!)
{
    try
    {
        int ordinal = reader.GetOrdinal(columnName);
        if (reader.IsDBNull(ordinal))
            return defaultValue;

        return (T)Convert.ChangeType(reader.GetValue(ordinal), typeof(T));
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
