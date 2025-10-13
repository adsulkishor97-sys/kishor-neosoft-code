 public async Task<IActionResult> GetAffiliateBenchmark(GetAffiliateBenchmarkRequest request)
 {

     var caseHierarchyResult = await _benchMarkServices.GetAffiliateBenchmark(request);
     if (caseHierarchyResult == null) return NoContent();
     else return Ok(caseHierarchyResult);
 }
 public class GetAffiliateBenchmarkRequest
{
    public string? affiliateId { get; set; }
    public string? kpiCode { get; set; }
    public string? startDate { get; set; }
    public string? endDate { get; set; }
}
 public class GetAffiliateBenchmarkResponse
 {
     public List<BenchmarkGroupedData> groupedBenchmarkData { get; set; } = new List<BenchmarkGroupedData>();

 }
 [ExcludeFromCodeCoverage]
 public class BenchmarkGroupedData
 {
     public string? affiliateName { get; set; }
     public long? target { get; set; } = 0;
     public int state { get; set; } = 0;
     public int? direction { get; set; } = 0;
     public decimal actual { get; set; } = 0;
     public decimal absolute { get; set; } = 0;
     public decimal bestAchievedEver { get; set; } = 0;
     public decimal bestAchievedEverMin { get; set; } = 0;
     public decimal bestAchievedForSinglePeriod { get; set; } = 0;



 }
 Task<List<BenchmarkGroupedData>> GetAffiliateBenchmark(GetAffiliateBenchmarkRequest request);
