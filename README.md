public class AssetBenchmarkService
{
    private readonly IBenchMarkRepository _benchMarkRepository;

    public AssetBenchmarkService(IBenchMarkRepository benchMarkRepository)
    {
        _benchMarkRepository = benchMarkRepository;
    }

    public async Task<List<AssetBenchmarkGroupedData>> GetAssetBenchmark(GetAssetBenchmarkRequest request)
    {
        var dateRange = FormatDatesAsset(request.startDate!, request.endDate!);
        var sapIds = FormatSapIds(request.sapId);
        var sqlKpiDataList = await _benchMarkRepository.GetBusinessKPIDetails();
        var kpiInfo = sqlKpiDataList.Find(x => x.kpiCode == request.kpiCode);

        var context = new AssetBenchmarkQueryContext
        {
            FormattedStartDate = dateRange.FormattedStartDate,
            FormattedEndDate = dateRange.FormattedEndDate,
            FormattedStartDateYMD = dateRange.FormattedStartDateYMD,
            FormattedEndDateYMD = dateRange.FormattedEndDateYMD,
            SapIds = sapIds,
            QuotedPmCode = ""
        };

        var groupResultData = await GetAssetBenchmarkDataAsync(request, context);

        if (groupResultData.Count == 0)
            return new List<AssetBenchmarkGroupedData>();

        if (!string.IsNullOrEmpty(request.kpiCode))
            AddAssetComputedFields(groupResultData, sqlKpiDataList, request.kpiCode, kpiInfo);

        return groupResultData;
    }

    private static (string FormattedStartDate, string FormattedEndDate, string FormattedStartDateYMD, string FormattedEndDateYMD)
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
        var sapList = sapIds?.Split(',').Select(int.Parse).ToList() ?? new List<int>();
        return string.Join(",", sapList.Select(x => x.ToString().Length < 18 ? $"'000000000{x}'" : $"'{x}'"));
    }

    private async Task<List<AssetBenchmarkGroupedData>> GetAssetBenchmarkDataAsync(
        GetAssetBenchmarkRequest request,
        AssetBenchmarkQueryContext context)
    {
        var assetQuery = new AssetBenchmarkQuery();
        var plantDataList = assetQuery.plantDataList
            .Where(p => p.kpiCode == request.kpiCode)
            .ToList();

        var tasks = plantDataList.Select(item => ExecutePlantQueryAsync(item, request, context));
        var results = await Task.WhenAll(tasks);

        return results.SelectMany(x => x).ToList();
    }

    private async Task<List<AssetBenchmarkGroupedData>> ExecutePlantQueryAsync(
        AssetBenchmarkPlantData item,
        GetAssetBenchmarkRequest request,
        AssetBenchmarkQueryContext context)
    {
        var queryParams = new AssetQueryParameters
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

        var query = ReplaceAssetQuery(queryParams);

        return await _benchMarkRepository.ExecuteBigDataQuery<AssetBenchmarkGroupedData>(
            query,
            reader => new AssetBenchmarkGroupedData
            {
                manufacturer = reader["manufacturer"] != DBNull.Value ? Convert.ToString(reader["manufacturer"]) : "",
                modelNumber = reader["model_number"] != DBNull.Value ? Convert.ToString(reader["model_number"]) : "",
                bestAchievedEver = reader["best_achieved_ever"] != DBNull.Value ? Convert.ToDecimal(reader["best_achieved_ever"]) : 0,
                absolute = reader["absolute"] != DBNull.Value ? Convert.ToDecimal(reader["absolute"]) : 0,
                actual = reader["actual"] != DBNull.Value ? Convert.ToInt32(reader["actual"]) : 0,
                sapId = reader["sap_id"] != DBNull.Value ? Convert.ToString(reader["sap_id"])!.TrimStart('0') : "",
            }) ?? new List<AssetBenchmarkGroupedData>();
    }

    private static void AddAssetComputedFields(
        List<AssetBenchmarkGroupedData> groupData,
        List<KpiDataJsonResponse> sqlKpiDataList,
        string kpiCode,
        KpiDataJsonResponse? kpiInfo)
    {
        if (!groupData.Any()) return;

        decimal bestAchievedEverMin = kpiInfo?.direction == 1 ? groupData.Min(x => x.bestAchievedEver) : groupData.Max(x => x.bestAchievedEver);
        decimal bestAchievedForSinglePeriod = kpiInfo?.direction == 1 ? groupData.Min(x => x.actual) : groupData.Max(x => x.actual);

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
}

// Helper DTO to reduce parameters
public class AssetBenchmarkQueryContext
{
    public string FormattedStartDate { get; set; } = "";
    public string FormattedEndDate { get; set; } = "";
    public string FormattedStartDateYMD { get; set; } = "";
    public string FormattedEndDateYMD { get; set; } = "";
    public string SapIds { get; set; } = "";
    public string QuotedPmCode { get; set; } = "";
}
