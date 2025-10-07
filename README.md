public async Task<IActionResult> GetKPITooltip(KpiInputRequest request)
{
    var getAffiliateCodeList = _configServices.GetAffiliateCodeList(request.affiliateId);
    GetCaseHierarchyRequest affiliateRequest = new()
    {
        bigDataAffiliateIdList = $"{string.Join(",", getAffiliateCodeList.Select(n => $"'{n}'"))}"
    };
    if (affiliateRequest.bigDataAffiliateIdList == null) return Unauthorized();
    var KpiToolTipResult = await _currentServices.GetKPITooltipAsyncNew(request, affiliateRequest.bigDataAffiliateIdList, request.startDate!, request.endDate!);
    if (KpiToolTipResult! == null) return NoContent();
    else return Ok(KpiToolTipResult);
}
  public class KpiInputRequest
 {
     public string? kpiCode { get; set; }
     public string? page { get; set; }
     public int? affiliateId { get; set; }
     public int? plantId { get; set; }
     public string? startDate { get; set; }
     public string? endDate { get; set; }
 }
public class KpiTooltipResponse
{
    public List<Label>? labels { get; set; }
    public KpiTooltipDetails? kpiTooltipDetails { get; set; }    
}
public class Label
{
    public string? title { get; set; }
    public string? uom { get; set; }
    public string? value { get; set; }
}
Task<KpiTooltipResponse> GetKPITooltipAsyncNew(KpiInputRequest request, string affiliateRequest, string startDate, string endDate);      
