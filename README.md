public async Task<List<AssetBenchmarkGroupedData>> GetAssetBenchmark(GetAssetBenchmarkRequest request)
{
    // Step 1: Validate input
    if (request == null) 
        return new List<AssetBenchmarkGroupedData>();

    // Step 2: Format dates & SAP IDs
    var (formattedStartDate, formattedEndDate, formattedStartDateYyyyMmDd, formattedEndDateYyyyMmDd) =
        FormatDatesAsset(request.startDate!, request.endDate!);
    var formattedSapIds = FormatSapIds(request.sapId);

    // Step 3: Fetch KPI details
    var sqlKpiDataList = await _benchMarkRepository.GetBusinessKPIDetails();
    var kpiInfo = sqlKpiDataList.Find(x => x.kpiCode == request.kpiCode);

    // Step 4: Retrieve benchmark data
    const string quotedPmCode = "";
    var groupResultData = await GetAssetBenchmarkDataAsync(
        request,
        formattedStartDate,
        formattedEndDate,
        formattedStartDateYyyyMmDd,
        formattedEndDateYyyyMmDd,
        formattedSapIds,
        quotedPmCode);

    // Step 5: Prepare and enrich results
    return PrepareFinalBenchmarkData(groupResultData, sqlKpiDataList, request.kpiCode, kpiInfo);
}

private static List<AssetBenchmarkGroupedData> PrepareFinalBenchmarkData(
    List<AssetBenchmarkGroupedData> groupData,
    List<KpiDataJsonResponse> sqlKpiDataList,
    string kpiCode,
    KpiDataJsonResponse? kpiInfo)
{
    if (groupData == null || groupData.Count == 0)
        return new List<AssetBenchmarkGroupedData>();

    if (!string.IsNullOrEmpty(kpiCode))
        AddAssetComputedFields(groupData, sqlKpiDataList, kpiCode, kpiInfo);

    return groupData;
}

private async Task<List<AssetBenchmarkGroupedData>> GetAssetBenchmarkDataAsync(
    GetAssetBenchmarkRequest request,
    string formattedStartDate,
    string formattedEndDate,
    string formattedStartDateYyyyMmDd,
    string formattedEndDateYyyyMmDd,
    string formattedSapIds,
    string quotedPmCode)
{
    var assetBenchmarkQuery = new AssetBenchmarkQuery();
    var queryList = assetBenchmarkQuery.plantDataList
        .Where(a => a.kpiCode == request.kpiCode)
        .ToList();

    var tasks = queryList.Select(item =>
        ExecuteBenchmarkQueryAsync(item, request, formattedStartDate, formattedEndDate,
                                   formattedStartDateYyyyMmDd, formattedEndDateYyyyMmDd,
                                   formattedSapIds, quotedPmCode));

    var results = await Task.WhenAll(tasks);
    return results.SelectMany(x => x).ToList();
}

private async Task<List<AssetBenchmarkGroupedData>> ExecuteBenchmarkQueryAsync(
    PlantDataItem item,
    GetAssetBenchmarkRequest request,
    string formattedStartDate,
    string formattedEndDate,
    string formattedStartDateYyyyMmDd,
    string formattedEndDateYyyyMmDd,
    string formattedSapIds,
    string quotedPmCode)
{
    var parameters = new AssetQueryParameters
    {
        overalN = item.bestAchievedEver ?? string.Empty,
        formattedStartDate = formattedStartDate,
        formattedEndDate = formattedEndDate,
        formattedStartDateYyyyMmDd = formattedStartDateYyyyMmDd,
        formattedEndDateYyyyMmDd = formattedEndDateYyyyMmDd,
        affiliateRequest = string.Empty,
        request = request,
        quotedPmCode = quotedPmCode,
        sapIds = formattedSapIds
    };

    var query = ReplaceAssetQuery(parameters);

    return await _benchMarkRepository.ExecuteBigDataQuery<AssetBenchmarkGroupedData>(
        query,
        MapReaderToAssetBenchmarkData
    ) ?? new List<AssetBenchmarkGroupedData>();
}

private static AssetBenchmarkGroupedData MapReaderToAssetBenchmarkData(IDataReader reader)
{
    return new AssetBenchmarkGroupedData
    {
        manufacturer = reader["manufacturer"]?.ToString() ?? string.Empty,
        modelNumber = reader["model_number"]?.ToString() ?? string.Empty,
        bestAchievedEver = reader["best_achieved_ever"] != DBNull.Value ? Convert.ToDecimal(reader["best_achieved_ever"]) : 0,
        absolute = reader["absolute"] != DBNull.Value ? Convert.ToDecimal(reader["absolute"]) : 0,
        actual = reader["actual"] != DBNull.Value ? Convert.ToInt32(reader["actual"]) : 0,
        sapId = reader["sap_id"]?.ToString()?.TrimStart('0') ?? string.Empty
    };
}

private static void AddAssetComputedFields(
    List<AssetBenchmarkGroupedData> groupData,
    List<KpiDataJsonResponse> sqlKpiDataList,
    string kpiCode,
    KpiDataJsonResponse? kpiInfo)
{
    if (groupData == null || !groupData.Any() || string.IsNullOrEmpty(kpiCode))
        return;

    var direction = kpiInfo?.direction ?? 0;
    var bestAchievedEverMin = direction == 1
        ? groupData.Min(x => x.bestAchievedEver)
        : groupData.Max(x => x.bestAchievedEver);

    var bestAchievedForSinglePeriod = direction == 1
        ? groupData.Min(x => x.actual)
        : groupData.Max(x => x.actual);

    var sqlDirection = sqlKpiDataList.FirstOrDefault(x => x.kpiCode == kpiCode)?.direction ?? 0;
    var targetMax = sqlKpiDataList.FirstOrDefault(x => x.kpiCode == kpiCode)?.overallTargetMax ?? 0;

    foreach (var item in groupData)
    {
        item.direction = sqlDirection;
        item.target = (int)targetMax;
        item.bestAchievedEverMin = bestAchievedEverMin;
        item.bestAchievedForSinglePeriod = bestAchievedForSinglePeriod;
    }
}

private static (string formattedStartDate, string formattedEndDate, string formattedStartDateYyyyMmDd, string formattedEndDateYyyyMmDd)
    FormatDatesAsset(string startDate, string endDate)
{
    var start = DateTime.Parse(startDate, CultureInfo.InvariantCulture);
    var end = DateTime.Parse(endDate, CultureInfo.InvariantCulture);

    return (
        start.ToString("yyyyMM"),
        end.ToString("yyyyMM"),
        start.ToString("yyyy-MM-dd"),
        end.ToString("yyyy-MM-dd")
    );
}

private static string FormatSapIds(string? sapIds)
{
    if (string.IsNullOrWhiteSpace(sapIds))
        return string.Empty;

    var sapList = sapIds.Split(',')
                        .Select(id => id.Trim())
                        .Where(id => !string.IsNullOrEmpty(id))
                        .Select(id =>
                        {
                            if (!long.TryParse(id, out _)) return string.Empty;
                            return id.Length < 18 ? $"'000000000{id}'" : $"'{id}'";
                        })
                        .Where(id => !string.IsNullOrEmpty(id))
                        .ToList();

    return string.Join(",", sapList);
}
