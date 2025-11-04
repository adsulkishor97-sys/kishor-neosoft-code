 private static (string formattedStartDate, string formattedEndDate, string startDateTimeDate, string endDateTimeDate)
ParseAndFormatDates(string startDate, string endDate)
 {
     var start = !string.IsNullOrWhiteSpace(startDate) ? DateTime.Parse(startDate, CultureInfo.InvariantCulture) : DateTime.Now;
     var end = !string.IsNullOrWhiteSpace(endDate) ? DateTime.Parse(endDate, CultureInfo.InvariantCulture) : DateTime.Now;
     return (start.ToString("yyyy-MM-dd"), end.ToString("yyyy-MM-dd"), start.ToString("yyyyMM"), end.ToString("yyyyMM"));
 }
