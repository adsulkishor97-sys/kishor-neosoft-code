    public async Task<List<PlantBenchmarkGroupedData>> GetPlantBenchmark(GetPlantBenchmarkRequest request)
    {
        var (formatedStartdate, formatedEnddate, formatedStartdateyymmdd, formatedEnddateyymmdd) = FormatDates(request);

        var plantIds = request.plantId?.Split(',').Select(int.Parse).ToList() ?? new List<int>();
        var formatPlantIds = string.Join(",", plantIds);
        var templateIdlist = request.assetClassId != 0 ? request.assetClassId.ToString() : string.Empty;

        var sqlKpiDataList = await _benchMarkRepository.GetBusinessKPIDetails();
        var kpiinfo = sqlKpiDataList.Find(x => x.kpiCode == request.kpiCode);

        var groupData = await GetBenchmarkDataAsync(request, formatedStartdate, formatedEnddate, formatedStartdateyymmdd, formatedEnddateyymmdd, formatPlantIds, templateIdlist);

        if (groupData == null || groupData.Count == 0)
            return CreateDefaultBenchmarkResult();

        AddComputedFields(groupData, sqlKpiDataList, request.kpiCode, kpiinfo);

        return groupData!;
    }
    private static (string, string, string, string) FormatDates(GetPlantBenchmarkRequest request)
    {
        var startDateTime = DateTime.Parse(request.startDate!, CultureInfo.InvariantCulture);
        var endDateTime = DateTime.Parse(request.endDate!, CultureInfo.InvariantCulture);

        var formatedStartdate = startDateTime.ToString("yyyyMM");
        var formatedEnddate = endDateTime.ToString("yyyyMM");
        var formatedStartdateyymmdd = startDateTime.ToString("yyyy-MM-dd");
        var formatedEnddateyymmdd = endDateTime.ToString("yyyy-MM-dd");

        return (formatedStartdate, formatedEnddate, formatedStartdateyymmdd, formatedEnddateyymmdd);
    }
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

            if (result != null && result.Count > 0)
                return result;
        }

        return new List<PlantBenchmarkGroupedData>();
    }
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
    private static void AddComputedFields(
        List<PlantBenchmarkGroupedData> groupData,
        List<SqlKpiData> sqlKpiDataList,
        string kpiCode,
        SqlKpiData? kpiinfo)
    {
        decimal bestAchivedEverMin, bestAchivedForSinglePeriod;

        if (kpiinfo?.direction == 1)
        {
            bestAchivedEverMin = groupData.Min(x => x.bestAchievedEver);
            bestAchivedForSinglePeriod = groupData.Min(x => x.actual);
        }
        else
        {
            bestAchivedEverMin = groupData.Max(x => x.bestAchievedEver);
            bestAchivedForSinglePeriod = groupData.Max(x => x.actual);
        }

        long? sqltarget = sqlKpiDataList
            .Where(x => x.kpiCode == kpiCode)
            .Select(x => x.overallTargetMax)
            .FirstOrDefault();

        int? sqldirection = sqlKpiDataList
            .Where(x => x.kpiCode == kpiCode)
            .Select(x => x.direction)
            .FirstOrDefault();

        foreach (var itemData in groupData)
        {
            itemData.bestAchievedEverMin = bestAchivedEverMin;
            itemData.bestAchievedForSinglePeriod = bestAchivedForSinglePeriod;
            itemData.target = sqltarget;
            itemData.direction = sqldirection;
        }
    }
 public async Task<List<KpiDataJsonResponse>> GetBusinessKPIDetails()
 {
     return await ExecuteQueryListAsync<KpiDataJsonResponse>
        (SPConstant.USP_UI_GET_BUSINESS_KPI_DETAILS,
        new
        {
        });
 }
