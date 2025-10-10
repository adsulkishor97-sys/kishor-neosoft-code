    public async Task<List<AssetBenchmarkGroupedData>> GetAssetBenchmark(GetAssetBenchmarkRequest request)
    {
        var (formatedStartdate, formatedEnddate, formatedStartdateyymmdd, formatedEnddateyymmdd) = FormatDatesAsset(request.startDate!, request.endDate!);
        var formatSapIds = FormatSapIds(request.sapId);
        var sqlKpiDataList = await _benchMarkRepository.GetBusinessKPIDetails();
        var kpiinfo = sqlKpiDataList.Find(x => x.kpiCode == request.kpiCode);
        string quatedpmcode = "";
        var groupresultdata = await GetAssetBenchmarkDataAsync(
            request,
            formatedStartdate,
            formatedEnddate,
            formatedStartdateyymmdd,
            formatedEnddateyymmdd,
            formatSapIds,
            quatedpmcode);

        if (groupresultdata == null|| groupresultdata.Count == 0)
            return new List<AssetBenchmarkGroupedData>();
        if (request != null && request.kpiCode != null)
        {
            AddAssetComputedFields(groupresultdata, sqlKpiDataList, request.kpiCode, kpiinfo);
        }
        return groupresultdata;
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

    private static void AddAssetComputedFields(
        List<AssetBenchmarkGroupedData> groupData,
        List<KpiDataJsonResponse> sqlKpiDataList,
        string kpiCode,
        KpiDataJsonResponse? kpiinfo)
    {
        decimal bestAchivedEverMin = kpiinfo?.direction == 1 ? groupData.Min(x => x.bestAchievedEver) : groupData.Max(x => x.bestAchievedEver);
        decimal bestAchivedForSinglePeriod = kpiinfo?.direction == 1 ? groupData.Min(x => x.actual) : groupData.Max(x => x.actual);

        int? sqldirection = sqlKpiDataList.Where(x => x.kpiCode == kpiCode).Select(x => x.direction).FirstOrDefault();
        long? targetMax = sqlKpiDataList.Where(x => x.kpiCode == kpiCode).Select(x => x.overallTargetMax).FirstOrDefault();

        foreach (var itemData in groupData)
        {
            itemData.direction = sqldirection ?? 0;
            itemData.target = (int)(targetMax ?? 0);
            itemData.bestAchievedEverMin = bestAchivedEverMin;
            itemData.bestAchievedForSinglePeriod = bestAchivedForSinglePeriod;
        }
    }
