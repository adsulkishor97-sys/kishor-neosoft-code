 private static DateTime GetParsedDate(string? inupuData)
        {
            return !string.IsNullOrWhiteSpace(inupuData)
                ? DateTime.Parse(inupuData, CultureInfo.InvariantCulture) : DateTime.Now;
        }
