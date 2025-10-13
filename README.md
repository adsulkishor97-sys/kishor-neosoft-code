public async Task<IActionResult> PerformanceSummary(GetKpiPerformanceRequest reportRequest)
{
    var getAffiliateCodeList = _configServices.GetAffiliateCodeList(reportRequest.affiliateId);
    GetCaseHierarchyRequest affiliateIdrequest = new()
    {
        bigDataAffiliateIdList = $"{string.Join(",", getAffiliateCodeList.Select(n => $"'{n}'"))}"
    };
    if (affiliateIdrequest.bigDataAffiliateIdList == null) return Unauthorized();
    var caseHierarchyResult = await _performanceSummServices.PerformanceSummaryAsync(reportRequest, affiliateIdrequest.bigDataAffiliateIdList);
    if (caseHierarchyResult! == null) return NoContent();
    else return Ok(caseHierarchyResult);
}
 public class GetKpiPerformanceRequest
 {
     public string? startDate { get; set; }
     public string? endDate { get; set; }
     public string? performanceSummary { get; set; }
     public int? affiliateId { get; set; }
     public int? plantId { get; set; }
 }
  public class KpiPerformanceResponse
 {
     public string? performanceSummary { get; set; } 
     public decimal average { get; set; }
     public decimal target { get; set; }
     public string? bestAffiliateName { get; set; }
     public decimal bestAffiliate { get; set; }
     public string? bestPlantName { get; set; }
     public decimal bestPlant { get; set; }
     public List<Affiliate>? affiliates { get; set; }
     public List<Plant>? plants { get; set; }
     public List<KpiDetail>? kpis { get; set; }
 }
 List<int> GetAffiliateCodeList(int? affiliateId);
 Task<KpiPerformanceResponse> PerformanceSummaryAsync(GetKpiPerformanceRequest request, string affiliateRequest);
