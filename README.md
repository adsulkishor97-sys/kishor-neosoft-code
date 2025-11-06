 public async Task<IActionResult> GetAffiliateCriteria(GetBenchmarkCriteriaListRequest request)
 {
     var affiliatecriteriaListResult = await _benchMarkServices.GetAffiliateCriteria(request);
     if (affiliatecriteriaListResult == null) return NoContent();
     else return Ok(affiliatecriteriaListResult);
 }
 public async Task<IActionResult> GetAffiliateBenchmarkComparision(AffiliateBenchmarkComparisionRequest affiliateBenchmarkComparisionRequest)
{
    var affiliateComparisionResult = await _benchMarkServices.GetAffiliateBenchmarkComparisionAsync(affiliateBenchmarkComparisionRequest);
    if (affiliateComparisionResult == null) return NoContent();
    else return Ok(affiliateComparisionResult);
}
 public class AffiliateBenchmarkComparisionRequest
 {
     public string? affiliateId { get; set; }
     public string? kpiCode { get; set; }
     public string? startDate { get; set; }
     public string? endDate { get; set; }       
 }
 public class GetBenchmarkCriteriaListRequest
{
    public string? page { get; set; }
}
