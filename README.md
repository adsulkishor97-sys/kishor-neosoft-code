public async Task<IActionResult> GetAssetBenchmarkComparision(AssetBenchmarkComparisionRequest assetBenchmarkComparisionRequest)
{
    var assetComparisionListResult = await _benchMarkServices.GetAssetBenchmarkComparisionAsync(assetBenchmarkComparisionRequest);
    if (assetComparisionListResult == null) return NoContent();
    else return Ok(assetComparisionListResult);
}
public class AssetBenchmarkComparisionRequest
{
    public string? sapId { get; set; }
    public string? kpiCode { get; set; }
    public string? startDate { get; set; }
    public string? endDate { get; set; }
}
 public class AssetBenchmarkComparisionResponse
 {
     public string? sapId { get; set; }
     public decimal? actual { get; set; }
     public decimal? absolute { get; set; }
     public string? modelNumber { get; set; }
     public string? companyName { get; set; }
 }
 Task<List<AssetBenchmarkComparisionResponse>> GetAssetBenchmarkComparisionAsync(AssetBenchmarkComparisionRequest assetBenchmarkComparisionRequest);
