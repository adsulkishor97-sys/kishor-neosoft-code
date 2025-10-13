public class QueryParameters
{
    public string BaseQuery { get; set; } = string.Empty;
    public string PmCodes { get; set; } = string.Empty;
    public string AffiliateRequest { get; set; } = string.Empty;
    public string FormattedStartDate { get; set; } = string.Empty;
    public string FormattedEndDate { get; set; } = string.Empty;
    public string StartDateTimeDate { get; set; } = string.Empty;
    public string EndDateTimeDate { get; set; } = string.Empty;
    public int PlantId { get; set; } = 0;
}
private static string PrepareQuery(QueryParameters parameters)
{
    return parameters.BaseQuery
        .Replace("pmcodes", parameters.PmCodes)
        .Replace("affiliateRequest", parameters.AffiliateRequest)
        .Replace("startDate", parameters.FormattedStartDate)
        .Replace("endDate", parameters.FormattedEndDate)
        .Replace("formatedStartdate", parameters.StartDateTimeDate)
        .Replace("formatedEnddate", parameters.EndDateTimeDate)
        .Replace("StartDateyyyymmdd", parameters.FormattedStartDate)
        .Replace("EndDateyyyymmdd", parameters.FormattedEndDate)
        .Replace("StartDateyyyymm", parameters.StartDateTimeDate)
        .Replace("EndDateyyyymm", parameters.EndDateTimeDate)
        .Replace("plantIdList", parameters.PlantId.ToString());
}


var firstQuery = jsonFileResponseList.FirstOrDefault()?.query ?? string.Empty;

var queryParams = new QueryParameters
{
    BaseQuery = firstQuery,
    PmCodes = pmcodes,
    AffiliateRequest = affiliateRequest,
    FormattedStartDate = formattedStartDate,
    FormattedEndDate = formattedEndDate,
    StartDateTimeDate = startDateTimeDate,
    EndDateTimeDate = endDateTimeDate,
    PlantId = request!.plantId!
};

var query = PrepareQuery(queryParams);
