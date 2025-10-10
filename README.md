public async Task<List<AssetBenchmarkGroupedData>> GetAssetBenchmark(GetAssetBenchmarkRequest request)
{
    var (formattedStartDate, formattedEndDate, formattedStartDateYyyyMmDd, formattedEndDateYyyyMmDd) =
        FormatDatesAsset(request.startDate!, request.endDate!);

    var formattedSapIds = FormatSapIds(request.sapId);
    var sqlKpiDataList = await _benchMarkRepository.GetBusinessKPIDetails();
    var kpiInfo = sqlKpiDataList.Find(x => x.kpiCode == request.kpiCode);
    string quotedPmCode = string.Empty;

    var groupResultData = await GetAssetBenchmarkDataAsync(
        request,
        formattedStartDate,
        formattedEndDate,
        formattedStartDateYyyyMmDd,
        formattedEndDateYyyyMmDd,
        formattedSapIds,
        quotedPmCode);

    if (groupResultData == null || groupResultData.Count == 0)
        return new List<AssetBenchmarkGroupedData>();

    if (!string.IsNullOrEmpty(request.kpiCode))
        AddAssetComputedFields(groupResultData, sqlKpiDataList, request.kpiCode, kpiInfo);

    return groupResultData;
}

private static (string, string, string, string) FormatDatesAsset(string startDate, string endDate)
{
    var startDateTime = DateTime.Parse(startDate, CultureInfo.InvariantCulture);
    var endDateTime = DateTime.Parse(endDate, CultureInfo.InvariantCulture);

    return (
        startDateTime.ToString("yyyyMM"),
        endDateTime.ToString("yyyyMM"),
        startDateTime.ToString("yyyy-MM-dd"),
        endDateTime.ToString("yyyy-MM-dd")
    );
}

private static string FormatSapIds(string? sapIds)
{
    var sapList = sapIds?.Split(',').Select(int.Parse).ToList() ?? new List<int>();
    return string.Join(",", sapList.Select(x => x.ToString().Length < 18 ? $"'000000000{x}'" : $"'{x}'"));
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
    var queryItems = assetBenchmarkQuery.plantDataList
        .Where(p => p.kpiCode == request.kpiCode)
        .ToList();

    var tasks = queryItems.Select(item =>
        FetchBenchmarkDataAsync(item, request,
            formattedStartDate, formattedEndDate,
            formattedStartDateYyyyMmDd, formattedEndDateYyyyMmDd,
            formattedSapIds, quotedPmCode));

    var results = await Task.WhenAll(tasks);
    return results.SelectMany(r => r).ToList();
}

private async Task<List<AssetBenchmarkGroupedData>> FetchBenchmarkDataAsync(
    AssetBenchmarkItem item,
    GetAssetBenchmarkRequest request,
    string formattedStartDate,
    string formattedEndDate,
    string formattedStartDateYyyyMmDd,
    string formattedEndDateYyyyMmDd,
    string formattedSapIds,
    string quotedPmCode)
{
    var parameters = BuildQueryParameters(item, request,
        formattedStartDate, formattedEndDate,
        formattedStartDateYyyyMmDd, formattedEndDateYyyyMmDd,
        formattedSapIds, quotedPmCode);

    var query = ReplaceAssetQuery(parameters);

    var result = await _benchMarkRepository.ExecuteBigDataQuery(query, MapReaderToAssetData);
    return result ?? new List<AssetBenchmarkGroupedData>();
}

private static AssetQueryParameters BuildQueryParameters(
    AssetBenchmarkItem item,
    GetAssetBenchmarkRequest request,
    string formattedStartDate,
    string formattedEndDate,
    string formattedStartDateYyyyMmDd,
    string formattedEndDateYyyyMmDd,
    string formattedSapIds,
    string quotedPmCode)
{
    return new AssetQueryParameters
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
}

private static AssetBenchmarkGroupedData MapReaderToAssetData(IDataReader reader)
{
    return new AssetBenchmarkGroupedData
    {
        manufacturer = SafeGetString(reader, "manufacturer"),
        modelNumber = SafeGetString(reader, "model_number"),
        bestAchievedEver = SafeGetDecimal(reader, "best_achieved_ever"),
        absolute = SafeGetDecimal(reader, "absolute"),
        actual = SafeGetInt(reader, "actual"),
        sapId = SafeGetString(reader, "sap_id").TrimStart('0')
    };
}

private static string SafeGetString(IDataReader reader, string column)
    => reader[column] != DBNull.Value ? Convert.ToString(reader[column]) ?? string.Empty : string.Empty;

private static decimal SafeGetDecimal(IDataReader reader, string column)
    => reader[column] != DBNull.Value ? Convert.ToDecimal(reader[column]) : 0;

private static int SafeGetInt(IDataReader reader, string column)
    => reader[column] != DBNull.Value ? Convert.ToInt32(reader[column]) : 0;

private static void AddAssetComputedFields(
    List<AssetBenchmarkGroupedData> groupData,
    List<KpiDataJsonResponse> sqlKpiDataList,
    string kpiCode,
    KpiDataJsonResponse? kpiInfo)
{
    decimal bestAchievedEverMin = kpiInfo?.direction == 1
        ? groupData.Min(x => x.bestAchievedEver)
        : groupData.Max(x => x.bestAchievedEver);

    decimal bestAchievedForSinglePeriod = kpiInfo?.direction == 1
        ? groupData.Min(x => x.actual)
        : groupData.Max(x => x.actual);

    int? sqlDirection = sqlKpiDataList
        .Where(x => x.kpiCode == kpiCode)
        .Select(x => x.direction)
        .FirstOrDefault();

    long? targetMax = sqlKpiDataList
        .Where(x => x.kpiCode == kpiCode)
        .Select(x => x.overallTargetMax)
        .FirstOrDefault();

    foreach (var item in groupData)
    {
        item.direction = sqlDirection ?? 0;
        item.target = (int)(targetMax ?? 0);
        item.bestAchievedEverMin = bestAchievedEverMin;
        item.bestAchievedForSinglePeriod = bestAchievedForSinglePeriod;
    }
}
