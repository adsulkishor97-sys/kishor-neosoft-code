public async Task<List<AssetBenchmarkGroupedData>> GetAssetBenchmark(GetAssetBenchmarkRequest request)
{
    var context = CreateAssetBenchmarkContext(request);
    var sqlKpiDataList = await _benchMarkRepository.GetBusinessKPIDetails();
    var kpiInfo = sqlKpiDataList.Find(x => x.kpiCode == request.kpiCode);

    var plantDataList = new AssetBenchmarkQuery()
        .plantDataList
        .Where(p => p.kpiCode == request.kpiCode)
        .ToList();

    var tasks = plantDataList.Select(p => ExecuteSinglePlantAsync(p, request, context));
    var results = await Task.WhenAll(tasks);

    var groupResultData = results.SelectMany(x => x).ToList();

    if (!groupResultData.Any()) 
        return new List<AssetBenchmarkGroupedData>();

    UpdateComputedFields(groupResultData, sqlKpiDataList, kpiInfo, request.kpiCode);
    return groupResultData;
}

// Context to reduce parameters
private static AssetBenchmarkQueryContext CreateAssetBenchmarkContext(GetAssetBenchmarkRequest request)
{
    var startDate = DateTime.Parse(request.startDate!, CultureInfo.InvariantCulture);
    var endDate = DateTime.Parse(request.endDate!, CultureInfo.InvariantCulture);
    var sapIds = request.sapId?.Split(',').Select(int.Parse).ToList() ?? new List<int>();
    var formattedSapIds = string.Join(",", sapIds.Select(x => x.ToString().Length < 18 ? $"'000000000{x}'" : $"'{x}'"));

    return new AssetBenchmarkQueryContext
    {
        FormattedStartDate = startDate.ToString("yyyyMM"),
        FormattedEndDate = endDate.ToString("yyyyMM"),
        FormattedStartDateYMD = startDate.ToString("yyyy-MM-dd"),
        FormattedEndDateYMD = endDate.ToString("yyyy-MM-dd"),
        SapIds = formattedSapIds,
        QuotedPmCode = ""
    };
}

// Execute query for one plant
private async Task<List<AssetBenchmarkGroupedData>> ExecuteSinglePlantAsync(
    AssetBenchmarkPlantData plantData,
    GetAssetBenchmarkRequest request,
    AssetBenchmarkQueryContext context)
{
    var query = ReplaceAssetQuery(plantData.bestAchievedEver!, request, context);
    return await _benchMarkRepository.ExecuteBigDataQuery<AssetBenchmarkGroupedData>(
        query,
        MapToAssetBenchmarkGroupedData
    ) ?? new List<AssetBenchmarkGroupedData>();
}

// Map DbDataReader to object
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

// Reduce parameters by using context object
private static string ReplaceAssetQuery(string overalN, GetAssetBenchmarkRequest request, AssetBenchmarkQueryContext context)
{
    return overalN!
        .Replace("StartDateyyyymmdd", context.FormattedStartDateYMD)
        .Replace("EndDateyyyymmdd", context.FormattedEndDateYMD)
        .Replace("affiliateRequest", "")
        .Replace("${startDateyyyymm}", $"'{context.FormattedStartDate}'")
        .Replace("${endDateyyyymm}", $"'{context.FormattedEndDate}'")
        .Replace("${startDate}", $"'{context.FormattedStartDate}'")
        .Replace("${endDate}", $"'{context.FormattedEndDate}'")
        .Replace("${sapId}", context.SapIds);
}

// Compute min/max and update fields
private static void UpdateComputedFields(
    List<AssetBenchmarkGroupedData> groupData,
    List<KpiDataJsonResponse> sqlKpiDataList,
    KpiDataJsonResponse? kpiInfo,
    string kpiCode)
{
    if (!groupData.Any()) return;

    decimal bestAchivedEverMin = kpiInfo?.direction == 1 ? groupData.Min(x => x.bestAchievedEver) : groupData.Max(x => x.bestAchievedEver);
    decimal bestAchivedForSinglePeriod = kpiInfo?.direction == 1 ? groupData.Min(x => x.actual) : groupData.Max(x => x.actual);

    var kpiData = sqlKpiDataList.Find(x => x.kpiCode == kpiCode);
    int direction = kpiData?.direction ?? 0;
    long targetMax = kpiData?.overallTargetMax ?? 0;

    foreach (var item in groupData)
    {
        item.direction = direction;
        item.target = (int)targetMax;
        item.bestAchievedEverMin = bestAchivedEverMin;
        item.bestAchievedForSinglePeriod = bestAchivedForSinglePeriod;
    }
}

// Context DTO
public class AssetBenchmarkQueryContext
{
    public string FormattedStartDate { get; set; } = "";
    public string FormattedEndDate { get; set; } = "";
    public string FormattedStartDateYMD { get; set; } = "";
    public string FormattedEndDateYMD { get; set; } = "";
    public string SapIds { get; set; } = "";
    public string QuotedPmCode { get; set; } = "";
}
