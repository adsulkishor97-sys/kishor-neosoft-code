// Parameter holder to reduce argument count
private class BenchmarkQueryContext
{
    public string FormattedStartMonth { get; set; } = string.Empty;
    public string FormattedEndMonth { get; set; } = string.Empty;
    public string FormattedStartDateYyyyMmDd { get; set; } = string.Empty;
    public string FormattedEndDateYyyyMmDd { get; set; } = string.Empty;
    public string FormattedSapIds { get; set; } = string.Empty;
    public string QuotedPmCode { get; set; } = string.Empty;
}

private async Task<List<AssetBenchmarkGroupedData>> GetAssetBenchmarkForItemAsync(
    AffiliateBenchmark item,
    GetAssetBenchmarkRequest request,
    BenchmarkQueryContext context)
{
    var parameters = new AssetQueryParameters
    {
        overalN = item.bestAchievedEver ?? string.Empty,
        formattedStartDate = context.FormattedStartMonth,
        formattedEndDate = context.FormattedEndMonth,
        formattedStartDateYyyyMmDd = context.FormattedStartDateYyyyMmDd,
        formattedEndDateYyyyMmDd = context.FormattedEndDateYyyyMmDd,
        affiliateRequest = string.Empty,
        request = request,
        quotedPmCode = context.QuotedPmCode,
        sapIds = context.FormattedSapIds
    };

    var query = ReplaceAssetQuery(parameters);

    return await _benchMarkRepository.ExecuteBigDataQuery<AssetBenchmarkGroupedData>(query, MapAssetBenchmarkGroupedData)
           ?? new List<AssetBenchmarkGroupedData>();
}
private async Task<List<AssetBenchmarkGroupedData>> GetAssetBenchmarkDataAsync(
    GetAssetBenchmarkRequest request,
    string formatedStartdate,
    string formatedEnddate,
    string formatedStartdateyymmdd,
    string formatedEnddateyymmdd,
    string formatSapIds,
    string quatedpmcode)
{
    var assetBenchmarkQuery = new AssetBenchmarkQuery();
    var plantDataList = assetBenchmarkQuery.plantDataList
        .Where(a => a.kpiCode == request.kpiCode)
        .ToList();

    var context = new BenchmarkQueryContext
    {
        FormattedStartMonth = formatedStartdate,
        FormattedEndMonth = formatedEnddate,
        FormattedStartDateYyyyMmDd = formatedStartdateyymmdd,
        FormattedEndDateYyyyMmDd = formatedEnddateyymmdd,
        FormattedSapIds = formatSapIds,
        QuotedPmCode = quatedpmcode
    };

    var tasks = plantDataList.Select(item =>
        GetAssetBenchmarkForItemAsync(item, request, context));

    var results = await Task.WhenAll(tasks);
    return results.SelectMany(x => x).ToList();
}
