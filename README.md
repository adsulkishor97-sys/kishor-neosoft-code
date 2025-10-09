public async Task<List<AssetBenchmarkComparisionResponse>> GetAssetBenchmarkComparisionAsync(
    AssetBenchmarkComparisionRequest assetBenchmarkComparisionRequest)
{
    var (assetResponseList, formatedSapIds) = PrepareAssetBenchmarkComparisionInputs(assetBenchmarkComparisionRequest);

    var results = await ExecuteAssetBenchmarkComparisionQueriesAsync(
        assetResponseList,
        assetBenchmarkComparisionRequest.kpiCode,
        assetBenchmarkComparisionRequest.startDate!,
        assetBenchmarkComparisionRequest.endDate!,
        formatedSapIds
    );

    return results;
}

#region Helper Methods

/// <summary>
/// Prepares input data (asset list and formatted SAP IDs) for the benchmark comparison.
/// </summary>
private static (List<BenchmarkComparision> assetResponseList, string formatedSapIds)
    PrepareAssetBenchmarkComparisionInputs(AssetBenchmarkComparisionRequest request)
{
    var assetBenchmarkComparisionQueries = new AssetBenchmarkComparisionQueries();
    var assetResponseList = assetBenchmarkComparisionQueries.assetDataList.ToList();

    var sapIds = request.sapId?.Split(',').Select(int.Parse).ToList() ?? new List<int>();
    var formatedSapIds = string.Join(",", sapIds.Select(x => x.ToString().Length < 18 ? $"'000000000{x}'" : $"'{x}'"));

    return (assetResponseList, formatedSapIds);
}

/// <summary>
/// Executes all asset benchmark comparison queries asynchronously and aggregates the results.
/// </summary>
private async Task<List<AssetBenchmarkComparisionResponse>> ExecuteAssetBenchmarkComparisionQueriesAsync(
    List<BenchmarkComparision> assetResponseList,
    string kpiCode,
    string startDate,
    string endDate,
    string formatedSapIds)
{
    var tasks = assetResponseList
        .Where(a => a.kpiCode == kpiCode)
        .Select(item => ExecuteSingleAssetBenchmarkQueryAsync(item, startDate, endDate, formatedSapIds))
        .ToList();

    var data = await Task.WhenAll(tasks);
    return data.SelectMany(x => x).ToList();
}

/// <summary>
/// Executes a single benchmark comparison query and returns mapped results.
/// </summary>
private async Task<List<AssetBenchmarkComparisionResponse>> ExecuteSingleAssetBenchmarkQueryAsync(
    BenchmarkComparision item,
    string startDate,
    string endDate,
    string formatedSapIds)
{
    var query = ReplaceQuery(item.query!, startDate, endDate, "", "", formatedSapIds, "");

    return await _benchMarkRepository.ExecuteBigDataQuery<AssetBenchmarkComparisionResponse>(
        query,
        reader => new AssetBenchmarkComparisionResponse
        {
            sapId = reader["sap_id"] != DBNull.Value ? reader["sap_id"].ToString() : "0",
            actual = reader["actual"] != DBNull.Value ? Convert.ToDecimal(reader["actual"]) : 0,
            absolute = reader["absolute"] != DBNull.Value ? Convert.ToDecimal(reader["absolute"]) : 0,
            modelNumber = reader["model_number"] != DBNull.Value ? reader["model_number"].ToString() : "0",
            companyName = reader["manufacturer"] != DBNull.Value ? reader["manufacturer"].ToString() : "0",
        }) ?? new List<AssetBenchmarkComparisionResponse>();
}

#endregion
