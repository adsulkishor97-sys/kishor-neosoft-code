public async Task<List<PlantBenchmarkGroupedData>> GetPlantBenchmark(GetPlantBenchmarkRequest request)
{
    // Step 1: Format the input dates
    var (formattedStartDate, formattedEndDate, formattedStartDateYyyyMmDd, formattedEndDateYyyyMmDd) = FormatDates(request);

    // Step 2: Prepare Plant and Template Data
    var (formatPlantIds, templateIdList) = PreparePlantAndTemplateData(request);

    // Step 3: Fetch KPI details
    var sqlKpiDataList = await _benchMarkRepository.GetBusinessKPIDetails();
    var kpiInfo = sqlKpiDataList.Find(x => x.kpiCode == request.kpiCode);

    // Step 4: Get Plant Benchmark Data
    var groupData = await GetBenchmarkDataAsync(
        request,
        formattedStartDate,
        formattedEndDate,
        formattedStartDateYyyyMmDd,
        formattedEndDateYyyyMmDd,
        formatPlantIds,
        templateIdList
    );

    // Step 5: Handle empty data case
    if (groupData == null || groupData.Count == 0)
        return CreateEmptyBenchmarkResult();

    // Step 6: Apply KPI calculations
    ApplyKpiMetrics(groupData, kpiInfo);

    // Step 7: Apply SQL metadata (target, direction)
    ApplySqlMetadata(groupData, sqlKpiDataList, request.kpiCode);

    // Step 8: Return final result
    return groupData;
}
private static (string, string, string, string) FormatDates(GetPlantBenchmarkRequest request)
{
    var startDateTime = DateTime.Parse(request.startDate!, CultureInfo.InvariantCulture);
    var endDateTime = DateTime.Parse(request.endDate!, CultureInfo.InvariantCulture);

    var formattedStartDate = startDateTime.ToString("yyyyMM");
    var formattedEndDate = endDateTime.ToString("yyyyMM");
    var formattedStartDateYyyyMmDd = startDateTime.ToString("yyyy-MM-dd");
    var formattedEndDateYyyyMmDd = endDateTime.ToString("yyyy-MM-dd");

    return (formattedStartDate, formattedEndDate, formattedStartDateYyyyMmDd, formattedEndDateYyyyMmDd);
}

private static (string formatPlantIds, string templateIdList) PreparePlantAndTemplateData(GetPlantBenchmarkRequest request)
{
    var plantIds = request.plantId?.Split(',').Select(int.Parse).ToList() ?? new List<int>();
    var formatPlantIds = string.Join(",", plantIds);
    var templateIdList = request.assetClassId != 0 ? request.assetClassId.ToString() : string.Empty;

    return (formatPlantIds, templateIdList);
}

private async Task<List<PlantBenchmarkGroupedData>> GetBenchmarkDataAsync(
    GetPlantBenchmarkRequest request,
    string formattedStartDate,
    string formattedEndDate,
    string formattedStartDateYyyyMmDd,
    string formattedEndDateYyyyMmDd,
    string formatPlantIds,
    string templateIdList)
{
    var plantBenchmarkQuery = new PlantBenchmarkQuery();
    var jsonFileGlobalResponseList = plantBenchmarkQuery.plantDataList.ToList();
    var quotedPmCode = string.Empty;

    foreach (var item in jsonFileGlobalResponseList.Where(a => a.kpiCode == request.kpiCode))
    {
        var parameters = new PlantQueryParameters
        {
            OveralN = item.bestAchievedEver ?? string.Empty,
            FormattedStartDate = formattedStartDate,
            FormattedEndDate = formattedEndDate,
            FormattedStartDateYyyyMmDd = formattedStartDateYyyyMmDd,
            FormattedEndDateYyyyMmDd = formattedEndDateYyyyMmDd,
            AffiliateRequest = string.Empty,
            Request = request,
            QuotedPmCode = quotedPmCode,
            PlantIds = formatPlantIds,
            TemplateIdList = templateIdList
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

        if (result != null && result.Count > 0)
            return result;
    }

    return new List<PlantBenchmarkGroupedData>();
}

private static void ApplyKpiMetrics(List<PlantBenchmarkGroupedData> groupData, SqlKpiInfo? kpiInfo)
{
    decimal bestAchievedEverValue;
    decimal bestAchievedForSinglePeriod;

    if (kpiInfo?.direction == 1)
    {
        bestAchievedEverValue = groupData.Min(x => x.bestAchievedEver);
        bestAchievedForSinglePeriod = groupData.Min(x => x.actual);
    }
    else
    {
        bestAchievedEverValue = groupData.Max(x => x.bestAchievedEver);
        bestAchievedForSinglePeriod = groupData.Max(x => x.actual);
    }

    foreach (var item in groupData)
    {
        item.bestAchievedEverMin = bestAchievedEverValue;
        item.bestAchievedForSinglePeriod = bestAchievedForSinglePeriod;
    }
}

private static void ApplySqlMetadata(List<PlantBenchmarkGroupedData> groupData, List<SqlKpiInfo> sqlKpiDataList, string kpiCode)
{
    var sqlTarget = sqlKpiDataList
        .Where(x => x.kpiCode == kpiCode)
        .Select(x => x.overallTargetMax)
        .FirstOrDefault();

    var sqlDirection = sqlKpiDataList
        .Where(x => x.kpiCode == kpiCode)
        .Select(x => x.direction)
        .FirstOrDefault();

    foreach (var item in groupData)
    {
        item.target = sqlTarget;
        item.direction = sqlDirection;
    }
}

private static List<PlantBenchmarkGroupedData> CreateEmptyBenchmarkResult()
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
