private async Task<List<AssetBenchmarkGroupedData>> GetAssetBenchmarkDataAsync(
    GetAssetBenchmarkRequest request,
    string formattedStartDate,
    string formattedEndDate,
    string formattedStartDateYyyyMmDd,
    string formattedEndDateYyyyMmDd,
    string formattedSapIds,
    string quotedPmCode)
{
    var assetBenchmarkQuery = new AssetBenchmarkQuery();
    var queryList = assetBenchmarkQuery.plantDataList
        .Where(a => a.kpiCode == request.kpiCode)
        .ToList();

    var tasks = queryList
        .AsParallel()
        .Select(item => ExecuteBenchmarkQueryAsync(item, request,
            formattedStartDate,
            formattedEndDate,
            formattedStartDateYyyyMmDd,
            formattedEndDateYyyyMmDd,
            formattedSapIds,
            quotedPmCode));

    var results = await Task.WhenAll(tasks);
    return results.SelectMany(x => x).ToList();
}

// 🔹 Extracted helper method: reduces nesting & improves readability
private async Task<List<AssetBenchmarkGroupedData>> ExecuteBenchmarkQueryAsync(
    PlantDataItem item,
    GetAssetBenchmarkRequest request,
    string formattedStartDate,
    string formattedEndDate,
    string formattedStartDateYyyyMmDd,
    string formattedEndDateYyyyMmDd,
    string formattedSapIds,
    string quotedPmCode)
{
    var parameters = new AssetQueryParameters
    {
        overalN = item.bestAchievedEver ?? string.Empty,
        formattedStartDate = formattedStartDate,
        formattedEndDate = formattedEndDate,
        formattedStartDateYyyyMmDd = formattedStartDateYyyyMmDd,
        formattedEndDateYyyyMmDd = formattedEndDateYyyyMmDd,
        affiliateRequest = string.Empty,
        request = request,
        quotedPmCode = quotedPmCode,
        sapIds = formattedSapIds
    };

    var query = ReplaceAssetQuery(parameters);

    return await _benchMarkRepository.ExecuteBigDataQuery<AssetBenchmarkGroupedData>(
        query,
        MapReaderToAssetBenchmarkData
    ) ?? new List<AssetBenchmarkGroupedData>();
}

// 🔹 Reader mapping extracted to a standalone method (removes inner lambda complexity)
private AssetBenchmarkGroupedData MapReaderToAssetBenchmarkData(IDataReader reader)
{
    return new AssetBenchmarkGroupedData
    {
        manufacturer = reader["manufacturer"]?.ToString() ?? string.Empty,
        modelNumber = reader["model_number"]?.ToString() ?? string.Empty,
        bestAchievedEver = reader["best_achieved_ever"] != DBNull.Value ? Convert.ToDecimal(reader["best_achieved_ever"]) : 0,
        absolute = reader["absolute"] != DBNull.Value ? Convert.ToDecimal(reader["absolute"]) : 0,
        actual = reader["actual"] != DBNull.Value ? Convert.ToInt32(reader["actual"]) : 0,
        sapId = reader["sap_id"]?.ToString()?.TrimStart('0') ?? string.Empty
    };
}
