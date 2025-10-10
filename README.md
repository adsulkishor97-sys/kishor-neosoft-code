private async Task<List<AssetBenchmarkGroupedData>> GetAssetBenchmarkDataAsync(
    GetAssetBenchmarkRequest request,
    string formatedStartdate,
    string formatedEnddate,
    string formatedStartdateyymmdd,
    string formatedEnddateyymmdd,
    string formatSapIds,
    string quatedpmcode)
{
    var assetBenchmarkQuery = new AssetBenchmarkQuery();
    var FileGlobalResponseList = assetBenchmarkQuery.plantDataList.ToList();

    var tasks = FileGlobalResponseList
        .AsParallel()
        .Where(a => a.kpiCode == request.kpiCode)
        .Select(async item =>
        {
            var parameters = new AssetQueryParameters
            {
                overalN = item.bestAchievedEver ?? string.Empty,
                formattedStartDate = formatedStartdate,
                formattedEndDate = formatedEnddate,
                formattedStartDateYyyyMmDd = formatedStartdateyymmdd,
                formattedEndDateYyyyMmDd = formatedEnddateyymmdd,
                affiliateRequest = string.Empty,
                request = request,
                quotedPmCode = quatedpmcode,
                sapIds = formatSapIds
            };

            var query = ReplaceAssetQuery(parameters);

            return await _benchMarkRepository.ExecuteBigDataQuery<AssetBenchmarkGroupedData>(
                query,
                reader => new AssetBenchmarkGroupedData
                {
                    manufacturer = reader["manufacturer"] != DBNull.Value ? Convert.ToString(reader["manufacturer"]) : "",
                    modelNumber = reader["model_number"] != DBNull.Value ? Convert.ToString(reader["model_number"]) : "",
                    bestAchievedEver = reader["best_achieved_ever"] != DBNull.Value ? Convert.ToDecimal(reader["best_achieved_ever"]) : 0,
                    absolute = reader["absolute"] != DBNull.Value ? Convert.ToDecimal(reader["absolute"]) : 0,
                    actual = reader["actual"] != DBNull.Value ? Convert.ToInt32(reader["actual"]) : 0,
                    sapId = reader["sap_id"] != DBNull.Value ? Convert.ToString(reader["sap_id"])!.TrimStart('0') : "",
                }) ?? new List<AssetBenchmarkGroupedData>();
        });

    var bestAchievedResults = await Task.WhenAll(tasks);
    return bestAchievedResults.SelectMany(x => x).ToList();
}
Refactor this method to reduce its Cognitive Complexity from 18 to the 15 allowed.

