public async Task<List<AssetBenchmarkGroupedData>> GetAssetBenchmark(GetAssetBenchmarkRequest request)
{
    // Step 1: Format dates and SAP IDs
    var (formattedStart, formattedEnd, formattedStartYMD, formattedEndYMD) = FormatDates(request.startDate!, request.endDate!);
    var formattedSapIds = FormatSapIds(request.sapId);

    // Step 2: Get KPI data
    var sqlKpiDataList = await _benchMarkRepository.GetBusinessKPIDetails();
    var kpiInfo = sqlKpiDataList.Find(x => x.kpiCode == request.kpiCode);

    // Step 3: Get plant data list
    var assetQuery = new AssetBenchmarkQuery();
    var plantDataList = assetQuery.plantDataList.ToList();

    // Step 4: Fetch benchmark data from DB
    var rawResults = await FetchBenchmarkDataAsync(plantDataList, request, formattedStart, formattedEnd, formattedStartYMD, formattedEndYMD, formattedSapIds);

    var groupResultData = rawResults.SelectMany(x => x).ToList();
    if (!groupResultData.Any()) return new List<AssetBenchmarkGroupedData>();

    // Step 5: Compute metrics
    AddBenchmarkMetrics(groupResultData, sqlKpiDataList, request.kpiCode, kpiInfo);

    return groupResultData;
}

#region Private Helpers

private static (string formattedStart, string formattedEnd, string formattedStartYMD, string formattedEndYMD) FormatDates(string startDate, string endDate)
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
    if (string.IsNullOrWhiteSpace(sapIds)) return string.Empty;
    var list = sapIds.Split(',').Select(id => id.Trim()).ToList();
    return string.Join(",", list.Select(x => x.Length < 18 ? $"'000000000{x}'" : $"'{x}'"));
}

private async Task<List<List<AssetBenchmarkGroupedData>>> FetchBenchmarkDataAsync(
    List<AssetBenchmarkItem> plantDataList,
    GetAssetBenchmarkRequest request,
    string formattedStart,
    string formattedEnd,
    string formattedStartYMD,
    string formattedEndYMD,
    string formattedSapIds)
{
    var tasks = plantDataList
        .Where(p => p.kpiCode == request.kpiCode)
        .Select(async item =>
        {
            var parameters = new AssetQueryParameters
            {
                overalN = item.bestAchievedEver ?? string.Empty,
                formattedStartDate = formattedStart,
                formattedEndDate = formattedEnd,
                formattedStartDateYyyyMmDd = formattedStartYMD,
                formattedEndDateYyyyMmDd = formattedEndYMD,
                affiliateRequest = string.Empty,
                request = request,
                quotedPmCode = string.Empty,
                sapIds = formattedSapIds
            };

            var query = ReplaceAssetQuery(parameters);

            return await _benchMarkRepository.ExecuteBigDataQuery<AssetBenchmarkGroupedData>(query, MapReaderToData)
                   ?? new List<AssetBenchmarkGroupedData>();
        });

    return await Task.WhenAll(tasks);
}

private static AssetBenchmarkGroupedData MapReaderToData(IDataReader reader)
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

private static void AddBenchmarkMetrics(List<AssetBenchmarkGroupedData> groupData, List<KpiDataJsonResponse> sqlKpiDataList, string kpiCode, KpiDataJsonResponse? kpiInfo)
{
    decimal bestEver = kpiInfo?.direction == 1
        ? groupData.Min(x => x.bestAchievedEver)
        : groupData.Max(x => x.bestAchievedEver);

    decimal bestPeriod = kpiInfo?.direction == 1
        ? groupData.Min(x => x.actual)
        : groupData.Max(x => x.actual);

    int direction = sqlKpiDataList.FirstOrDefault(x => x.kpiCode == kpiCode)?.direction ?? 0;
    int target = (int)(sqlKpiDataList.FirstOrDefault(x => x.kpiCode == kpiCode)?.overallTargetMax ?? 0);

    foreach (var item in groupData)
    {
        item.direction = direction;
        item.target = target;
        item.bestAchievedEverMin = bestEver;
        item.bestAchievedForSinglePeriod = bestPeriod;
    }
}

private static string ReplaceAssetQuery(AssetQueryParameters parameters)
{
    return parameters.overalN!
        .Replace("StartDateyyyymmdd", parameters.formattedStartDateYyyyMmDd)
        .Replace("EndDateyyyymmdd", parameters.formattedEndDateYyyyMmDd)
        .Replace("affiliateRequest", parameters.affiliateRequest)
        .Replace("${startDateyyyymm}", $"'{parameters.formattedStartDate}'")
        .Replace("${endDateyyyymm}", $"'{parameters.formattedEndDate}'")
        .Replace("${startDate}", $"'{parameters.formattedStartDate}'")
        .Replace("${endDate}", $"'{parameters.formattedEndDate}'")
        .Replace("${sapId}", parameters.sapIds);
}

#endregion
