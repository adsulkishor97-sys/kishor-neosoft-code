private static (string, string, string, string) FormatDatesAffiliate(GetAffiliateBenchmarkRequest request)
{
    // Define allowed formats
    string[] formats = { "dd-MM-yyyy", "MM-dd-yyyy", "yyyy-MM-dd", "dd/MM/yyyy", "MM/dd/yyyy" };

    if (!DateTime.TryParseExact(request.startDate, formats, CultureInfo.InvariantCulture, DateTimeStyles.None, out var startDateTime))
        throw new FormatException($"Invalid start date format: {request.startDate}");

    if (!DateTime.TryParseExact(request.endDate, formats, CultureInfo.InvariantCulture, DateTimeStyles.None, out var endDateTime))
        throw new FormatException($"Invalid end date format: {request.endDate}");

    var formattedStartdate = startDateTime.ToString("yyyyMM");
    var formattedEnddate = endDateTime.ToString("yyyyMM");
    var formattedStartdateYMD = startDateTime.ToString("yyyy-MM-dd");
    var formattedEnddateYMD = endDateTime.ToString("yyyy-MM-dd");

    return (formattedStartdate, formattedEnddate, formattedStartdateYMD, formattedEnddateYMD);
}
