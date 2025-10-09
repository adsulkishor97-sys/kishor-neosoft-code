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


            var tasks = FileGlobalResponseList
                .AsParallel()
                .Where(a => a.kpiCode == request.kpiCode)
                .Select(async item =>
                {
                    var queryBestAchievedEver = ReplaceAssetQuery(item.bestAchievedEver!, formatedStartdate, formatedEnddate, formatedStartdateyymmdd, formatedEnddateyymmdd, "", request, quatedpmcode, formatSapIds);
                    var bestAchievedresult = await _benchMarkRepository.ExecuteBigDataQuery<AssetBenchmarkGroupedData>(queryBestAchievedEver, reader => new AssetBenchmarkGroupedData
                    {
                        manufacturer = reader["manufacturer"] != DBNull.Value ? Convert.ToString(reader["manufacturer"]) : "",
                        modelNumber = reader["model_number"] != DBNull.Value ? Convert.ToString(reader["model_number"]) : "",
                        bestAchievedEver = reader["best_achieved_ever"] != DBNull.Value ? Convert.ToDecimal(reader["best_achieved_ever"]) : 0,
                        absolute = reader["absolute"] != DBNull.Value ? Convert.ToDecimal(reader["absolute"]) : 0,
                        actual = reader["actual"] != DBNull.Value ? Convert.ToInt32(reader["actual"]) : 0,
                        sapId = reader["sap_id"] != DBNull.Value ? Convert.ToString(reader["sap_id"])!.TrimStart('0') : "",

                    }) ?? new List<AssetBenchmarkGroupedData>();
                    return bestAchievedresult;


                });
