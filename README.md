private static string ReplaceQuery(string? query, string startDate, string endDate, string affiliateRequest1, string plantIds, string sapIds, string templateId)
{
    //convert string to datetime
    var startDateTime = DateTime.Parse(startDate, CultureInfo.InvariantCulture);
    var endDateTime = DateTime.Parse(endDate, CultureInfo.InvariantCulture);

    //convert format as yyyymm 202508
    var formatedStartdate = "'" + startDateTime.ToString("yyyyMM") + "'";
    var formatedEnddate = "'" + endDateTime.ToString("yyyyMM") + "'";

    //convert format as yyyymmdd 20250801
    var formatedStartdateyyyymmdd = "'" + startDateTime.ToString("yyyyMMdd") + "'";
    var formatedEnddateyyyymmdd = "'" + endDateTime.ToString("yyyyMMdd") + "'";

    var finalQuery = query!
                    .Replace("${affiliateIdList}", affiliateRequest1)
                    .Replace("${startDate}", "'" + startDateTime.ToString("yyyy-MM-dd") + "'")
                    .Replace("${endDate}", "'" + endDateTime.ToString("yyyy-MM-dd") + "'")
                    .Replace("${startDateyyyymmdd}", formatedStartdateyyyymmdd)
                    .Replace("${endDateyyyymmdd}", formatedEnddateyyyymmdd)
                    .Replace("${plantIdList}", plantIds)
                    .Replace("${startDateyyyymm}", formatedStartdate)
                    .Replace("${endDateyyyymm}", formatedEnddate)
                    .Replace("${sapId}", sapIds)
                    .Replace("${templateId}", templateId);
    return finalQuery;
}
