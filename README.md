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
public class AffiliateBenchmarkComparisionResponse
{
    public string? affiliateName { get; set; }
    public decimal? actual { get; set; }
    public decimal? absolute { get; set; }
}
Task<List<AffiliateBenchmarkComparisionResponse>> GetAffiliateBenchmarkComparisionAsync(AffiliateBenchmarkComparisionRequest affiliateBenchmarkComparisionRequest);
