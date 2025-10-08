public async Task<IActionResult> GetAffiliateCriteria(GetBenchmarkCriteriaListRequest request)
{
    var affiliatecriteriaListResult = await _benchMarkServices.GetAffiliateCriteria(request);
    if (affiliatecriteriaListResult == null) return NoContent();
    else return Ok(affiliatecriteriaListResult);
}
public class GetBenchmarkCriteriaListRequest
{
    public string? page { get; set; }
}
public class GetBenchmarkCriteriaListResponse
{
    public string? kpiType { get; set; }
    public List<BenchmarkCriteriaKpiList> data { get; set; } = new List<BenchmarkCriteriaKpiList>();
}
public class BenchmarkCriteriaKpiList
{
    public string? kpiCode { get; set; }
    public string? kpiName { get; set; }
}
Task<List<GetBenchmarkCriteriaListResponse>> GetAffiliateCriteria(GetBenchmarkCriteriaListRequest request);
