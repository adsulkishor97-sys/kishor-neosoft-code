public async Task<List<BenchmarkGroupedData>> GetAffiliateBenchmark(GetAffiliateBenchmarkRequest request)
{
    List<BenchmarkGroupedData> groupData = new List<BenchmarkGroupedData>();
    //convert string to datetime
    var startDateTime = DateTime.Parse(request.startDate!, CultureInfo.InvariantCulture);
    var endDateTime = DateTime.Parse(request.endDate!, CultureInfo.InvariantCulture);

    //convert format as yyyymm 202508
    var formatedStartdate = startDateTime.ToString("yyyyMM");
    var formatedEnddate = endDateTime.ToString("yyyyMM");


    //convert format as yyyy-mm-dd 2025-08-01
   var  formatedStartdateyymmdd = startDateTime.ToString("yyyy-MM-dd");
   var formatedEnddateyymmdd = endDateTime.ToString("yyyy-MM-dd");

    AffiliateBenchmarkQuery affiliatebenchmarkquery = new AffiliateBenchmarkQuery(); 
    List<AffiliateBenchmark> GlobalResponseList = affiliatebenchmarkquery.affiliateDataList.ToList();
    var sqlKpiDataList = await _benchMarkRepository.GetBusinessKPIDetails();
    var kpiinfo = sqlKpiDataList.Find(x => x.kpiCode == request.kpiCode);
    var sqlAffiliateName = await _benchMarkRepository.GetAffiliateLists();
    var affiliateds = request.affiliateId?.Split(',').Select(int.Parse).ToList();

    var affiliateList = new List<GetCaseHierarchyResponse>();
    foreach (var item in affiliateds!)
    {
        var lstPlants = await _benchMarkRepository.GetCaseHierarchyAsync(item);
        affiliateList.AddRange(lstPlants);
    }


    foreach (var item in GlobalResponseList.Where(a => a.kpiCode == request.kpiCode))
    {
        var actual = AffilateReplaceQuery(item.query!, formatedStartdateyymmdd, formatedEnddateyymmdd, formatedStartdate, formatedEnddate, "", request);
        var queryBestAchievedEver = AffilateReplaceQuery(item.bestAchievedEver!, formatedStartdateyymmdd, formatedEnddateyymmdd, formatedStartdate, formatedEnddate, "", request);


        var actualQuery = await _benchMarkRepository.ExecuteBigDataQuery<BenchmarkGroupedData>(actual, reader =>
        {
            int affilateIdFromBigData = reader["affiliate_id"] != DBNull.Value ? Convert.ToInt32(reader["affiliate_id"]) : 0;
            string? affilateName = sqlAffiliateName.Find(x => x.affiliateId == affilateIdFromBigData)?.affiliateName;
            return new BenchmarkGroupedData
            {
               
                affiliateName = affilateName,
                actual = reader["actual"] != DBNull.Value ? Convert.ToDecimal(reader["actual"]) : 0,
                absolute = reader["absolute"] != DBNull.Value ? Convert.ToDecimal(reader["absolute"]) : 0,                        
            };
        }) ?? new List<BenchmarkGroupedData>();
        var bestAchievedResult = await _benchMarkRepository.ExecuteBigDataQuery<BenchmarkGroupedData>(queryBestAchievedEver, reader =>
        {
            int affilateIdFromBigData = reader["affiliate_id"] != DBNull.Value ? Convert.ToInt32(reader["affiliate_id"]) : 0;
            string? affilateName = sqlAffiliateName.Find(x => x.affiliateId == affilateIdFromBigData)?.affiliateName;
            return new BenchmarkGroupedData
            {
                bestAchievedEver = reader["best_achieved"] != DBNull.Value ? Convert.ToDecimal(reader["best_achieved"]) : 0,
                affiliateName = affilateName
            };
        }) ?? new List<BenchmarkGroupedData>();

         groupData = (from a in actualQuery
                      join b in bestAchievedResult
                         on a.affiliateName equals b.affiliateName into temp
                         from b in temp.DefaultIfEmpty()
                         select new BenchmarkGroupedData
                         {
                             affiliateName= a.affiliateName,
                             actual= a.actual,
                             absolute= a.absolute,
                             bestAchievedEver=b?.bestAchievedEver ?? 0
                         }).GroupBy(x=>x.affiliateName).Select(g=> new BenchmarkGroupedData
                         { 
                             affiliateName=g.Key,
                             actual=g.Sum(x=>x.actual),
                             absolute= g.Sum(x=>x.absolute),
                             bestAchievedEver=g.Max(x=>x.bestAchievedEver)
                         }
                         ).ToList();


        decimal bestAchivedEverMin;
        decimal bestAchivedForSinglePeriod;

            if (kpiinfo?.direction == 1)
            {
            bestAchivedEverMin = (groupData != null && groupData.Count > 0) ? groupData.Min(x => x.bestAchievedEverMin) : 0;
            bestAchivedForSinglePeriod = (groupData != null && groupData.Count > 0) ? groupData.Min(x => x.actual) : 0;
            
            }
            else
            {
                bestAchivedEverMin = groupData.Max(x => x.bestAchievedEver);
                bestAchivedForSinglePeriod = groupData.Max(x => x.actual);
            }


        if (actualQuery == null || actualQuery.Count == 0)
        {
            return new List<BenchmarkGroupedData>();
        }
        if (queryBestAchievedEver == null || queryBestAchievedEver.Length == 0)
        {
            return new List<BenchmarkGroupedData>();
        }




             int? sqlDirection = sqlKpiDataList.Where(x => x.kpiCode == request.kpiCode).Select(x => x.direction).FirstOrDefault();
            long? targetMin = sqlKpiDataList.Where(x => x.kpiCode == request.kpiCode).Select(x => x.overallTargetMin).FirstOrDefault();
            long? targetMax = sqlKpiDataList.Where(x => x.kpiCode == request.kpiCode).Select(x => x.overallTargetMax).FirstOrDefault();
            foreach (var itemData in groupData!)
            {

                if (itemData.actual >= targetMin && itemData.actual <= targetMax)
                {
                    itemData.state = 1;
                }
                else
                {
                    itemData.state = 0;
                }


                itemData.direction = sqlDirection;
                itemData.target = targetMax!;
                itemData.bestAchievedEverMin = bestAchivedEverMin;
                itemData.bestAchievedForSinglePeriod = bestAchivedForSinglePeriod;

            }

        

    }


    return groupData!;
}


public async Task<List<BenchmarkGroupedData>> GetAffiliateBenchmark(GetAffiliateBenchmarkRequest request)
{
    // Step 1: Prepare dates and basic data
    var (formatedStartdate, formatedEnddate, formatedStartdateyymmdd, formatedEnddateyymmdd) = FormatDatesAffiliate(request);
    var sqlKpiDataList = await _benchMarkRepository.GetBusinessKPIDetails();
    var kpiInfo = sqlKpiDataList.Find(x => x.kpiCode == request.kpiCode);
    var sqlAffiliateName = (await _benchMarkRepository.GetAffiliateLists())
        .Select(a => new AffiliateListResponse
        {
            affiliateId = a.affiliateId,
            affiliateName = a.affiliateName
        }).ToList();
    

    // Step 2: Get benchmark query data
    var groupData = await ProcessAffiliateBenchmarkAsync(
        request, formatedStartdate, formatedEnddate, formatedStartdateyymmdd, formatedEnddateyymmdd,
        sqlKpiDataList, sqlAffiliateName, kpiInfo
    );

    return groupData!;
}

#region Helper Methods

private static (string, string, string, string) FormatDatesAffiliate(GetAffiliateBenchmarkRequest request)
{
    var startDateTime = DateTime.Parse(request.startDate!, CultureInfo.InvariantCulture);
    var endDateTime = DateTime.Parse(request.endDate!, CultureInfo.InvariantCulture);

    var formatedStartdate = startDateTime.ToString("yyyyMM");
    var formatedEnddate = endDateTime.ToString("yyyyMM");
    var formatedStartdateyymmdd = startDateTime.ToString("yyyy-MM-dd");
    var formatedEnddateyymmdd = endDateTime.ToString("yyyy-MM-dd");

    return (formatedStartdate, formatedEnddate, formatedStartdateyymmdd, formatedEnddateyymmdd);
}

private async Task<List<BenchmarkGroupedData>> ProcessAffiliateBenchmarkAsync(
    GetAffiliateBenchmarkRequest request,
    string formatedStartdate,
    string formatedEnddate,
    string formatedStartdateyymmdd,
    string formatedEnddateyymmdd,
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
            item, request, formatedStartdate, formatedEnddate,
            formatedStartdateyymmdd, formatedEnddateyymmdd,
            sqlAffiliateName, sqlKpiDataList, kpiInfo
        );
    }

    return groupData;
}

private async Task<List<BenchmarkGroupedData>> ExecuteAffiliateBenchmarkQueriesAsync(
    AffiliateBenchmark item,
    GetAffiliateBenchmarkRequest request,
    string formatedStartdate,
    string formatedEnddate,
    string formatedStartdateyymmdd,
    string formatedEnddateyymmdd,
    List<AffiliateListResponse> sqlAffiliateName,
    List<KpiDataJsonResponse> sqlKpiDataList,
    KpiDataJsonResponse? kpiInfo)
{
    var actualQueryStr = AffilateReplaceQuery(item.query!, formatedStartdateyymmdd, formatedEnddateyymmdd, formatedStartdate, formatedEnddate, "", request);
    var bestQueryStr = AffilateReplaceQuery(item.bestAchievedEver!, formatedStartdateyymmdd, formatedEnddateyymmdd, formatedStartdate, formatedEnddate, "", request);

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

private static T GetValueOrDefault<T>(IDataRecord reader, string columnName, T defaultValue = default!)
{
    try
    {
        int ordinal = reader.GetOrdinal(columnName);
        return reader.IsDBNull(ordinal) ? defaultValue : (T)Convert.ChangeType(reader.GetValue(ordinal), typeof(T));
    }
    catch (IndexOutOfRangeException)
    {
        // Column not found
        return defaultValue;
    }
    catch (InvalidCastException)
    {
        // Type mismatch
        return defaultValue;
    }
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
    decimal bestAchivedEverMin, bestAchivedForSinglePeriod;

    if (kpiInfo?.direction == 1)
    {
        bestAchivedEverMin = groupData.Any() ? groupData.Min(x => x.bestAchievedEver) : 0;
        bestAchivedForSinglePeriod = groupData.Any() ? groupData.Min(x => x.actual) : 0;
    }
    else
    {
        bestAchivedEverMin = groupData.Any() ? groupData.Max(x => x.bestAchievedEver) : 0;
        bestAchivedForSinglePeriod = groupData.Any() ? groupData.Max(x => x.actual) : 0;
    }

    var sqlDirection = sqlKpiDataList.FirstOrDefault(x => x.kpiCode == kpiCode)?.direction;
    var targetMin = sqlKpiDataList.FirstOrDefault(x => x.kpiCode == kpiCode)?.overallTargetMin;
    var targetMax = sqlKpiDataList.FirstOrDefault(x => x.kpiCode == kpiCode)?.overallTargetMax;

    foreach (var item in groupData)
    {
        item.state = (item.actual >= targetMin && item.actual <= targetMax) ? 1 : 0;
        item.direction = sqlDirection;
        item.target = targetMax ?? 0;
        item.bestAchievedEverMin = bestAchivedEverMin;
        item.bestAchievedForSinglePeriod = bestAchivedForSinglePeriod;
    }
}

#endregion
