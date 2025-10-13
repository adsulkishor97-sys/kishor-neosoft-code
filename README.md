private static string PrepareQuery(string baseQuery, string pmcodes, string affiliateRequest, string formattedStartDate,
  string formattedEndDate, string startDateTimeDate, string endDateTimeDate, int? plantId)
{
    return baseQuery
    .Replace("pmcodes", pmcodes)
     .Replace("affiliateRequest", affiliateRequest)
     .Replace("startDate", formattedStartDate)
     .Replace("endDate", formattedEndDate)
     .Replace("formatedStartdate", startDateTimeDate)
     .Replace("formatedEnddate", endDateTimeDate)
     .Replace("StartDateyyyymmdd", formattedStartDate)
     .Replace("EndDateyyyymmdd", formattedEndDate)
     .Replace("StartDateyyyymm", startDateTimeDate)
     .Replace("EndDateyyyymm", endDateTimeDate)
     .Replace("plantIdList", plantId.ToString() ?? "0");
}
#calling point#
    var query = PrepareQuery(jsonFileResponseList.FirstOrDefault()?.query ?? string.Empty, pmcodes, affiliateRequest,
formattedStartDate, formattedEndDate, startDateTimeDate, endDateTimeDate, request!.plantId!);
