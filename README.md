private static void AddComputedFields(
    List<PlantBenchmarkGroupedData> groupData,
    List<KpiDataJsonResponse> sqlKpiDataList,
    string kpiCode,
    KpiDataJsonResponse? kpiinfo)
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
var sqlKpiDataListJson = await _benchMarkRepository.GetBusinessKPIDetails();
var sqlKpiDataList = sqlKpiDataListJson.Select(x => new SqlKpiData
{
    kpiCode = x.kpiCode,
    overallTargetMax = x.overallTargetMax,
    direction = x.direction
}).ToList();
