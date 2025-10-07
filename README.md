 public async Task<IActionResult> GetKpiData(GetKpiDetailsRequest request)
 {
     var getAffiliateCodeList = _configServices.GetAffiliateCodeList(request.affiliateId);
     GetCaseHierarchyRequest affiliateRequest = new()
     {
         bigDataAffiliateIdList = $"{string.Join(",", getAffiliateCodeList.Select(n => $"'{n}'"))}"
     };
     if (affiliateRequest.bigDataAffiliateIdList == null) return Unauthorized();

     var caseHierarchyResult = await _currentServices.GetKpiDetailsAsync(request.page!, affiliateRequest.bigDataAffiliateIdList,
         request.startDate!, request.endDate!, request.plantId!);
     if (caseHierarchyResult == null) return NoContent();
     else return Ok(caseHierarchyResult);
 }
 public class GetKpiDetailsRequest
 {
     public int? affiliateId { get; set; }
     public string? plantId { get; set; } = string.Empty;
     public string? page { get; set; }
     public string? startDate { get; set; }
     public string? endDate { get; set; }
 }
public class GetKpiDataResponse
{
    public int? plantId { get; set; }public string? kpiName { get; set; } public string? kpiCode { get; set; }
    public long? overallActual { get; set; }public string? overallTargetDisplay { get; set; }public long? overallTargetMin { get; set; }
    public long? overallTargetMax { get; set; }public int overallState { get; set; } = 0;public long? criticalActual { get; set; } 
    public string? criticalTargetDisplay { get; set; }public long? criticalTargetMin { get; set; }public long? criticalTargetMax { get; set; } 
    public int criticalState { get; set; } = 0;public string? subCategory { get; set; }
}
 public List<int> GetAffiliateCodeList(int? affiliateId)
 {
     // get AffilateCodes based on affilateId
     var lstAffiliates = _configRepository.GetAffiliateLists().Result;
     var getAffilateCodes = lstAffiliates.Where(a => affiliateId == null || a.affiliateId == affiliateId).Select(x => x.affiliateCode).ToList();
     return getAffilateCodes;
 }
  Task<List<GetKpiDataResponse>> GetKpiDetailsAsync(string page, string affiliateRequest, string startDate, string endDate, string affiliatePlantId = "");
