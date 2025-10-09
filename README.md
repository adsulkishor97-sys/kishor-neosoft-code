public async Task<List<AssetBenchmarkComparisionResponse>> GetAssetBenchmarkComparisionAsync(AssetBenchmarkComparisionRequest assetBenchmarkComparisionRequest)
{
    AssetBenchmarkComparisionQueries assetBenchmarkComparisionQueries = new AssetBenchmarkComparisionQueries();

    List<BenchmarkComparision> assetResponseList = assetBenchmarkComparisionQueries.assetDataList.ToList();

    var sapIds = assetBenchmarkComparisionRequest.sapId?.Split(',').Select(int.Parse).ToList();
    var formatedSapIds = string.Join(",", sapIds!.Select(x => x.ToString().Length < 18 ? $"'000000000{x}'" : $"'{x}'"));
    var tasks = assetResponseList
    .Where(a => a.kpiCode == assetBenchmarkComparisionRequest.kpiCode)
    .Select(async item =>
    {
        var query = ReplaceQuery(item.query!, assetBenchmarkComparisionRequest.startDate!, assetBenchmarkComparisionRequest.endDate!, "", "", formatedSapIds,"");

        var result = await _benchMarkRepository.ExecuteBigDataQuery<AssetBenchmarkComparisionResponse>(query, reader => new AssetBenchmarkComparisionResponse
        {
            sapId = reader["sap_id"] != DBNull.Value ? reader["sap_id"].ToString() : "0",
            actual = reader["actual"] != DBNull.Value ? Convert.ToDecimal(reader["actual"]) : 0,
            absolute = reader["absolute"] != DBNull.Value ? Convert.ToDecimal(reader["absolute"]) : 0,
            modelNumber = reader["model_number"] != DBNull.Value ? reader["model_number"].ToString() : "0",
            companyName = reader["manufacturer"] != DBNull.Value ? reader["manufacturer"].ToString() : "0",
            
        }) ?? new List<AssetBenchmarkComparisionResponse>();
        return result;
    }).ToList();
    var data = (await Task.WhenAll(tasks!)).ToList();
    var finalResult = data.SelectMany(x => x).ToList();
    return finalResult!;
}
