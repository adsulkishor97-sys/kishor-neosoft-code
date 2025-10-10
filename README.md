private async Task<List<AssetBenchmarkGroupedData>> GetAssetBenchmarkDataAsync(
    GetAssetBenchmarkRequest request,
    AssetBenchmarkQueryContext context)
{
    var assetBenchmarkQuery = new AssetBenchmarkQuery();
    var plantDataList = assetBenchmarkQuery.plantDataList
        .Where(p => p.kpiCode == request.kpiCode)
        .ToList();

    var tasks = plantDataList.Select(item => ExecuteSinglePlantQueryAsync(item, request, context));
    var results = await Task.WhenAll(tasks);

    return results.SelectMany(x => x).ToList();
}

private async Task<List<AssetBenchmarkGroupedData>> ExecuteSinglePlantQueryAsync(
    AssetBenchmarkPlantData item,
    GetAssetBenchmarkRequest request,
    AssetBenchmarkQueryContext context)
{
    var queryParams = CreateQueryParameters(item, request, context);
    var query = ReplaceAssetQuery(queryParams);

    return await _benchMarkRepository.ExecuteBigDataQuery<AssetBenchmarkGroupedData>(
        query,
        MapToAssetBenchmarkGroupedData
    ) ?? new List<AssetBenchmarkGroupedData>();
}

private static AssetQueryParameters CreateQueryParameters(
    AssetBenchmarkPlantData item,
    GetAssetBenchmarkRequest request,
    AssetBenchmarkQueryContext context)
{
    return new AssetQueryParameters
    {
        overalN = item.bestAchievedEver ?? string.Empty,
        formattedStartDate = context.FormattedStartDate,
        formattedEndDate = context.FormattedEndDate,
        formattedStartDateYyyyMmDd = context.FormattedStartDateYMD,
        formattedEndDateYyyyMmDd = context.FormattedEndDateYMD,
        affiliateRequest = string.Empty,
        request = request,
        quotedPmCode = context.QuotedPmCode,
        sapIds = context.SapIds
    };
}

private static AssetBenchmarkGroupedData MapToAssetBenchmarkGroupedData(DbDataReader reader)
{
    return new AssetBenchmarkGroupedData
    {
        manufacturer = reader["manufacturer"] != DBNull.Value ? Convert.ToString(reader["manufacturer"]) : "",
        modelNumber = reader["model_number"] != DBNull.Value ? Convert.ToString(reader["model_number"]) : "",
        bestAchievedEver = reader["best_achieved_ever"] != DBNull.Value ? Convert.ToDecimal(reader["best_achieved_ever"]) : 0,
        absolute = reader["absolute"] != DBNull.Value ? Convert.ToDecimal(reader["absolute"]) : 0,
        actual = reader["actual"] != DBNull.Value ? Convert.ToInt32(reader["actual"]) : 0,
        sapId = reader["sap_id"] != DBNull.Value ? Convert.ToString(reader["sap_id"])!.TrimStart('0') : "",
    };
}

// Helper DTO to wrap method parameters
public class AssetBenchmarkQueryContext
{
    public string FormattedStartDate { get; set; } = "";
    public string FormattedEndDate { get; set; } = "";
    public string FormattedStartDateYMD { get; set; } = "";
    public string FormattedEndDateYMD { get; set; } = "";
    public string SapIds { get; set; } = "";
    public string QuotedPmCode { get; set; } = "";
}
