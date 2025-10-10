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
    var actualQueryStr = AffilateReplaceQuery(
        item.query!,
        formattedStartdateYMD, formattedEnddateYMD,
        formattedStartdate, formattedEnddate,
        "",
        request
    );

    var bestQueryStr = AffilateReplaceQuery(
        item.bestAchievedEver!,
        formattedStartdateYMD, formattedEnddateYMD,
        formattedStartdate, formattedEnddate,
        "",
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
Method has 10 parameters, which is greater than the 7 authorized.

