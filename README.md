public async Task<List<PlantBenchmarkGroupedData>> GetPlantBenchmark(GetPlantBenchmarkRequest request)
{
    // 1. Format Dates
    var (formatedStartdate, formatedEnddate, formatedStartdateyymmdd, formatedEnddateyymmdd) = FormatDates(request);

    // 2. Format PlantIds & templateId
    var formatPlantIds = FormatPlantIds(request.plantId);
    var templateIdlist = request.assetClassId != 0 ? request.assetClassId.ToString() : string.Empty;

    // 3. Get KPI details
    var sqlKpiDataList = await _benchMarkRepository.GetBusinessKPIDetails();
    var kpiinfo = sqlKpiDataList.Find(x => x.kpiCode == request.kpiCode);

    // 4. Fetch Benchmark data
    var groupData = await GetBenchmarkDataAsync(
        request,
        formatedStartdate,
        formatedEnddate,
        formatedStartdateyymmdd,
        formatedEnddateyymmdd,
        formatPlantIds,
        templateIdlist);

    // 5. Return default if no data
    if (groupData == null || groupData.Count == 0)
        return CreateDefaultBenchmarkResult();

    // 6. Add computed fields
    AddComputedFields(groupData, sqlKpiDataList, request.kpiCode, kpiinfo);

    return groupData!;
}

// Helper: Format dates
private static (string, string, string, string) FormatDates(GetPlantBenchmarkRequest request)
{
    var startDateTime = DateTime.Parse(request.startDate!, CultureInfo.InvariantCulture);
    var endDateTime = DateTime.Parse(request.endDate!, CultureInfo.InvariantCulture);

    return (
        startDateTime.ToString("yyyyMM"),
        endDateTime.ToString("yyyyMM"),
        startDateTime.ToString("yyyy-MM-dd"),
        endDateTime.ToString("yyyy-MM-dd")
    );
}

// Helper: Format PlantIds string
private static string FormatPlantIds(string? plantId)
{
    var plantIds = plantId?.Split(',').Select(int.Parse).ToList() ?? new List<int>();
    return string.Join(",", plantIds);
}

// Helper: Fetch Benchmark data
private async Task<List<PlantBenchmarkGroupedData>> GetBenchmarkDataAsync(
    GetPlantBenchmarkRequest request,
    string formatedStartdate,
    string formatedEnddate,
    string formatedStartdateyymmdd,
    string formatedEnddateyymmdd,
    string formatPlantIds,
    string templateIdlist)
{
    var plantBenchmarkQuery = new PlantBenchmarkQuery();
    var jsonFileGlobalResponseList = plantBenchmarkQuery.plantDataList.ToList();
    var quatedpmcode = string.Empty;

    foreach (var item in jsonFileGlobalResponseList.Where(a => a.kpiCode == request.kpiCode))
    {
        var parameters = new PlantQueryParameters
        {
            OveralN = item.bestAchievedEver ?? string.Empty,
            FormattedStartDate = formatedStartdate,
            FormattedEndDate = formatedEnddate,
            FormattedStartDateYyyyMmDd = formatedStartdateyymmdd,
            FormattedEndDateYyyyMmDd = formatedEnddateyymmdd,
            AffiliateRequest = string.Empty,
            Request = request,
            QuotedPmCode = quatedpmcode,
            PlantIds = formatPlantIds,
            TemplateIdList = templateIdlist
        };

        var query = ReplacePlantQuery(parameters);

        var result = await _benchMarkRepository.ExecuteBigDataQuery<PlantBenchmarkGroupedData>(
            query,
            reader => new PlantBenchmarkGroupedData
            {
                plantId = reader["plant_id"] != DBNull.Value ? Convert.ToInt32(reader["plant_id"]) : 0,
                bestAchievedEver = reader["best_achieved"] != DBNull.Value ? Convert.ToDecimal(reader["best_achieved"]) : 0,
                absolute = reader["absolute"] != DBNull.Value ? Convert.ToDecimal(reader["absolute"]) : 0,
                actual = reader["actual"] != DBNull.Value ? Convert.ToDecimal(reader["actual"]) : 0
            }) ?? new List<PlantBenchmarkGroupedData>();

        if (result.Count > 0)
            return result;
    }

    return new List<PlantBenchmarkGroupedData>();
}

// Helper: Default result
private static List<PlantBenchmarkGroupedData> CreateDefaultBenchmarkResult()
{
    return new List<PlantBenchmarkGroupedData>
    {
        new PlantBenchmarkGroupedData
        {
            plantId = 0,
            bestAchievedEver = 0,
            absolute = 0,
            actual = 0
        }
    };
}

// Helper: Add computed fields
private static void AddComputedFields(
    List<PlantBenchmarkGroupedData> groupData,
    List<KpiDataJsonResponse> sqlKpiDataList,
    string kpiCode,
    KpiDataJsonResponse? kpiinfo)
{
    decimal bestAchivedEverMin = kpiinfo?.direction == 1 ? groupData.Min(x => x.bestAchievedEver) : groupData.Max(x => x.bestAchievedEver);
    decimal bestAchivedForSinglePeriod = kpiinfo?.direction == 1 ? groupData.Min(x => x.actual) : groupData.Max(x => x.actual);

    long? sqltarget = sqlKpiDataList.Where(x => x.kpiCode == kpiCode).Select(x => x.overallTargetMax).FirstOrDefault();
    int? sqldirection = sqlKpiDataList.Where(x => x.kpiCode == kpiCode).Select(x => x.direction).FirstOrDefault();

    foreach (var itemData in groupData)
    {
        itemData.bestAchievedEverMin = bestAchivedEverMin;
        itemData.bestAchievedForSinglePeriod = bestAchivedForSinglePeriod;
        itemData.target = sqltarget;
        itemData.direction = sqldirection;
    }
}
