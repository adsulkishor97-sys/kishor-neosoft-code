public async Task<List<BenchmarkGroupedData>> GetAffiliateBenchmark(GetAffiliateBenchmarkRequest request)
{
    var (formattedStartdate, formattedEnddate, formattedStartdateYMD, formattedEnddateYMD) = FormatDatesAffiliate(request);

    var sqlKpiDataList = await _benchMarkRepository.GetBusinessKPIDetails();
    var kpiInfo = sqlKpiDataList.Find(x => x.kpiCode == request.kpiCode);

    var sqlAffiliateName = (await _benchMarkRepository.GetAffiliateLists())
        .Select(a => new AffiliateListResponse
        {
            affiliateId = a.affiliateId,
            affiliateName = a.affiliateName
        }).ToList();
    var groupData = await ProcessAffiliateBenchmarkAsync(
        request,
        formattedStartdate, formattedEnddate,
        formattedStartdateYMD, formattedEnddateYMD,
        sqlKpiDataList, sqlAffiliateName, kpiInfo
    );

    return groupData!;
}

private static (string, string, string, string) FormatDatesAffiliate(GetAffiliateBenchmarkRequest request)
{
    var startDateTime = DateTime.Parse(request.startDate!, CultureInfo.InvariantCulture);
    var endDateTime = DateTime.Parse(request.endDate!, CultureInfo.InvariantCulture);

    var formattedStartdate = startDateTime.ToString("yyyyMM");
    var formattedEnddate = endDateTime.ToString("yyyyMM");
    var formattedStartdateYMD = startDateTime.ToString("yyyy-MM-dd");
    var formattedEnddateYMD = endDateTime.ToString("yyyy-MM-dd");

    return (formattedStartdate, formattedEnddate, formattedStartdateYMD, formattedEnddateYMD);
}

private async Task<List<BenchmarkGroupedData>> ProcessAffiliateBenchmarkAsync(
    GetAffiliateBenchmarkRequest request,
    string formattedStartdate,
    string formattedEnddate,
    string formattedStartdateYMD,
    string formattedEnddateYMD,
    List<KpiDataJsonResponse> sqlKpiDataList,
    List<AffiliateListResponse> sqlAffiliateName,
    KpiDataJsonResponse? kpiInfo)
{
    var affiliateBenchmarkQuery = new AffiliateBenchmarkQuery();
    var responseList = affiliateBenchmarkQuery.affiliateDataList
        .Where(a => a.kpiCode == request.kpiCode)
        .ToList();

    List<BenchmarkGroupedData> groupData = new();

    foreach (var item in responseList)
    {
        groupData = await ExecuteAffiliateBenchmarkQueriesAsync(
            item,
            request,
            formattedStartdate, formattedEnddate,
            formattedStartdateYMD, formattedEnddateYMD,
            sqlAffiliateName, sqlKpiDataList, kpiInfo
        );
    }

    return groupData;
}

private async Task<List<BenchmarkGroupedData>> ExecuteAffiliateBenchmarkQueriesAsync(
    AffiliateBenchmark item,
    GetAffiliateBenchmarkRequest request,
    string formattedStartdate,
    string formattedEnddate,
    string formattedStartdateYMD,
    string formattedEnddateYMD,
    List<AffiliateListResponse> sqlAffiliateName,
    List<KpiDataJsonResponse> sqlKpiDataList,
    KpiDataJsonResponse? kpiInfo)
{
    var actualQueryStr = AffilateReplaceQuery(item.query!,formattedStartdateYMD, formattedEnddateYMD,
        formattedStartdate, formattedEnddate,  request  );

    var bestQueryStr = AffilateReplaceQuery(
        item.bestAchievedEver!,
        formattedStartdateYMD, formattedEnddateYMD,
        formattedStartdate, formattedEnddate,             
        request
    );

    var actualQuery = await ExecuteAffiliateDataQueryAsync(actualQueryStr, sqlAffiliateName);
    var bestQuery = await ExecuteAffiliateDataQueryAsync(bestQueryStr, sqlAffiliateName, isBest: true);

    if (actualQuery.Count == 0 || bestQueryStr.Length == 0)
        return new List<BenchmarkGroupedData>();

    var mergedData = MergeAffiliateBenchmarkData(actualQuery, bestQuery);
    if (request != null && request.kpiCode != null)
    {
        UpdateComputedFields(mergedData, sqlKpiDataList, kpiInfo, request.kpiCode);
    }
    return mergedData;
}

private async Task<List<BenchmarkGroupedData>> ExecuteAffiliateDataQueryAsync(
    string query,
    List<AffiliateListResponse> sqlAffiliateName,
    bool isBest = false)
{
    return await _benchMarkRepository.ExecuteBigDataQuery<BenchmarkGroupedData>(
        query,
        reader =>
        {
            int affiliateId = GetValueOrDefault(reader, "affiliate_id", 0);
            string? affiliateName = sqlAffiliateName.Find(x => x.affiliateId == affiliateId)?.affiliateName;

            return isBest
                ? new BenchmarkGroupedData
                {
                    affiliateName = affiliateName,
                    bestAchievedEver = GetValueOrDefault(reader, "best_achieved", 0m)
                }
                : new BenchmarkGroupedData
                {
                    affiliateName = affiliateName,
                    actual = GetValueOrDefault(reader, "actual", 0m),
                    absolute = GetValueOrDefault(reader, "absolute", 0m)
                };
        }) ?? new List<BenchmarkGroupedData>();
}

private static List<BenchmarkGroupedData> MergeAffiliateBenchmarkData(
    List<BenchmarkGroupedData> actualData,
    List<BenchmarkGroupedData> bestData)
{
    return (from a in actualData
            join b in bestData on a.affiliateName equals b.affiliateName into temp
            from b in temp.DefaultIfEmpty()
            select new BenchmarkGroupedData
            {
                affiliateName = a.affiliateName,
                actual = a.actual,
                absolute = a.absolute,
                bestAchievedEver = b?.bestAchievedEver ?? 0
            })
           .GroupBy(x => x.affiliateName)
           .Select(g => new BenchmarkGroupedData
           {
               affiliateName = g.Key,
               actual = g.Sum(x => x.actual),
               absolute = g.Sum(x => x.absolute),
               bestAchievedEver = g.Max(x => x.bestAchievedEver)
           })
           .ToList();
}

private static void UpdateComputedFields(
    List<BenchmarkGroupedData> groupData,
    List<KpiDataJsonResponse> sqlKpiDataList,
    KpiDataJsonResponse? kpiInfo,
    string kpiCode)
{
    decimal bestAchievedEverMin, bestAchievedForSinglePeriod;

    if (kpiInfo?.direction == 1)
    {
        bestAchievedEverMin = groupData.Count > 0 ? groupData.Min(x => x.bestAchievedEver) : 0;
        bestAchievedForSinglePeriod = groupData.Count > 0 ? groupData.Min(x => x.actual) : 0;
    }
    else
    {
        bestAchievedEverMin = groupData.Count > 0 ? groupData.Max(x => x.bestAchievedEver) : 0;
        bestAchievedForSinglePeriod = groupData.Count > 0 ? groupData.Max(x => x.actual) : 0;
    }

    var sqlDirection = sqlKpiDataList.First(x => x.kpiCode == kpiCode)?.direction;
    var targetMin = sqlKpiDataList.First(x => x.kpiCode == kpiCode)?.overallTargetMin;
    var targetMax = sqlKpiDataList.First(x => x.kpiCode == kpiCode)?.overallTargetMax;

    foreach (var item in groupData)
    {
        item.state = (item.actual >= targetMin && item.actual <= targetMax) ? 1 : 0;
        item.direction = sqlDirection;
        item.target = targetMax ?? 0;
        item.bestAchievedEverMin = bestAchievedEverMin;
        item.bestAchievedForSinglePeriod = bestAchievedForSinglePeriod;
    }
}

private static T GetValueOrDefault<T>(DbDataReader reader, string columnName, T defaultValue = default!)
{
    try
    {
        int ordinal = reader.GetOrdinal(columnName);
        if (reader.IsDBNull(ordinal))
            return defaultValue;

        return (T)Convert.ChangeType(reader.GetValue(ordinal), typeof(T));
    }
    catch (IndexOutOfRangeException)
    {
        return defaultValue;
    }
    catch (InvalidCastException)
    {
        return defaultValue;
    }
}
