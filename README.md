public async Task<IActionResult> GetAssetBenchmark(GetAssetBenchmarkRequest request)
{
    var caseHierarchyResult = await _benchMarkServices.GetAssetBenchmark(request);
    if (caseHierarchyResult == null) return NoContent();
    else return Ok(caseHierarchyResult);
}
 public class GetAssetBenchmarkRequest
 {
     public string? sapId { get; set; }
     public string? kpiCode { get; set; }
     public string startDate { get; set; } = string.Empty;
     public string endDate { get; set; } = string.Empty;
 }
 public class GetAssetBenchmarkResponse
{
    public List<AssetBenchmarkGroupedData> assetgroupedBenchmarkData { get; set; } = new List<AssetBenchmarkGroupedData>();
}

public class AssetBenchmarkGroupedData
{
    public string? sapId { get; set; } = string.Empty;
    public int target { get; set; } = 0;
    public int direction { get; set; } = 0;
    public int actual { get; set; }
    public decimal? absolute { get; set; }
    public decimal bestAchievedEver { get; set; } = 0;
    public decimal bestAchievedEverMin { get; set; } = 0;
    public decimal bestAchievedForSinglePeriod { get; set; } = 0;
    public string? modelNumber { get; set; } = string.Empty;
    public string? manufacturer { get; set; } = string.Empty;
    public string? designValue { get; set; } = string.Empty;
}
Task<List<AssetBenchmarkGroupedData>> GetAssetBenchmark(GetAssetBenchmarkRequest request);
