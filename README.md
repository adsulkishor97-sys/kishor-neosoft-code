public class AssetQueryParameters
{
    public string OveralN { get; set; } = string.Empty;
    public string FormattedStartDate { get; set; } = string.Empty;
    public string FormattedEndDate { get; set; } = string.Empty;
    public string FormattedStartDateYyyyMmDd { get; set; } = string.Empty;
    public string FormattedEndDateYyyyMmDd { get; set; } = string.Empty;
    public string AffiliateRequest { get; set; } = string.Empty;
    public GetAssetBenchmarkRequest Request { get; set; } = default!;
    public string QuotedPmCode { get; set; } = string.Empty;
    public string SapIds { get; set; } = string.Empty;
}
public static string ReplaceAssetQuery(AssetQueryParameters parameters)
{
    var query = parameters.OveralN!
        .Replace("StartDateyyyymmdd", parameters.FormattedStartDateYyyyMmDd)
        .Replace("EndDateyyyymmdd", parameters.FormattedEndDateYyyyMmDd)
        .Replace("affiliateRequest", parameters.AffiliateRequest)
        .Replace("${startDateyyyymm}", $"'{parameters.FormattedStartDate}'")
        .Replace("${endDateyyyymm}", $"'{parameters.FormattedEndDate}'")
        .Replace("${startDate}", $"'{parameters.FormattedStartDate}'")
        .Replace("${endDate}", $"'{parameters.FormattedEndDate}'")
        .Replace("${sapId}", parameters.SapIds);

    return query;
}
var query = ReplaceAssetQuery(new AssetQueryParameters
{
    OveralN = overalN,
    FormattedStartDate = startDate,
    FormattedEndDate = endDate,
    FormattedStartDateYyyyMmDd = startYyyymmdd,
    FormattedEndDateYyyyMmDd = endYyyymmdd,
    AffiliateRequest = affiliateRequest,
    Request = assetRequest,
    QuotedPmCode = pmCode,
    SapIds = sapIds
});

