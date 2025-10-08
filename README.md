public async Task<IActionResult> GetAffiliateTrend(AffiliateTrendRequest request)
{
    var result = await _benchMarkServices.GetAffiliateTrend(request);
    if (result == null) return NoContent();
    else return Ok(result);
}
public class AffiliateTrendRequest
{
    public string? affiliateId { get; set; }
    public int kpiCode { get; set; }
    public string startDate { get; set; }=string.Empty;
    public string endDate { get; set; } = string.Empty;        

}
public class AffiliateTrendResponse
{
    public string? affiliateName { get; set; }
    public decimal actual { get; set; }
    public decimal absolute { get; set; }
    public string? time { get; set; }
    public string? frequency { get; set; }

}
Task<List<AffiliateTrendResponse>> GetAffiliateTrend(AffiliateTrendRequest request);
