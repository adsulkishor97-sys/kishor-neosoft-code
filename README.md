public async Task<IActionResult> GetAssetPerformanceCriteria(GetAssetPerformanceRequest request)
{
    var PlantListResult = await _benchMarkServices.GetAssetPerformanceCriteria(request);
    if (PlantListResult == null) return NoContent();
    else return Ok(PlantListResult);
}
public class GetAssetPerformanceRequest
{
    public string? sapId { get; set; }
}
public class AssetPerformanceCriteriaResponse
{
    public string? kpiType { get; set; }
    public List<BenchmarkCriteriaKpiList> data { get; set; } = new List<BenchmarkCriteriaKpiList>();
}
 public class BenchmarkCriteriaKpiList
 {
     public string? kpiCode { get; set; }
     public string? kpiName { get; set; }
     public string? absoluteUnit { get; set; }
     public string? actualUnit { get; set; }

 }
 Task<List<AssetPerformanceCriteriaResponse>> GetAssetPerformanceCriteria(GetAssetPerformanceRequest request);
