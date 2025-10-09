public class PlantQueryParameters
{
    public string OveralN { get; set; } = string.Empty;
    public string FormattedStartDate { get; set; } = string.Empty;
    public string FormattedEndDate { get; set; } = string.Empty;
    public string FormattedStartDateYyyyMmDd { get; set; } = string.Empty;
    public string FormattedEndDateYyyyMmDd { get; set; } = string.Empty;
    public string AffiliateRequest { get; set; } = string.Empty;
    public GetPlantBenchmarkRequest Request { get; set; } = default!;
    public string QuotedPmCode { get; set; } = string.Empty;
    public string PlantIds { get; set; } = string.Empty;
    public string TemplateIdList { get; set; } = string.Empty;
}
public static string ReplacePlantQuery(PlantQueryParameters parameters)
{
    var query = parameters.OveralN!
        .Replace("affiliateRequest1", parameters.AffiliateRequest)
        .Replace("StartDateyyyymmdd", parameters.FormattedStartDateYyyyMmDd)
        .Replace("EndDateyyyymmdd", parameters.FormattedEndDateYyyyMmDd)
        .Replace("affiliateRequest", parameters.AffiliateRequest)
        .Replace("${plantIdList}", parameters.Request.plantId)
        .Replace("plantIdList", parameters.Request.plantId)
        .Replace("StartDateyyyymm", parameters.FormattedStartDate)
        .Replace("${startDateyyyymm}", $"'{parameters.FormattedStartDate}'")
        .Replace("${endDateyyyymm}", $"'{parameters.FormattedEndDate}'")
        .Replace("EndDateyyyymm", parameters.FormattedEndDate)
        .Replace("quatedpmcode", parameters.QuotedPmCode)
        .Replace("${templateId}", parameters.TemplateIdList)
        .Replace("${startDate}", $"'{parameters.FormattedStartDate}'")
        .Replace("${endDate}", $"'{parameters.FormattedEndDate}'");

    return query;
}
var parameters = new PlantQueryParameters
{
    OveralN = item.bestAchievedEver ?? string.Empty,
    FormattedStartDate = formatedStartdate,
    FormattedEndDate = formatedEnddate,
    FormattedStartDateYyyyMmDd = formatedStartdateyymmdd,
    FormattedEndDateYyyyMmDd = formatedEnddateyymmdd,
    AffiliateRequest = string.Empty,
    Request = request,
    QuotedPmCode = quatedpmcode,
    PlantIds = formatPlantIds,
    TemplateIdList = templateIdlist
};

var queryBestAchievedEver = ReplacePlantQuery(parameters);
