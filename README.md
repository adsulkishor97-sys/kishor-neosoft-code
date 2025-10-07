public async Task<IActionResult> GetAffiliatePlantDistribution(GetAffiliatePlantDistributionRequest request)
{
    var getAffiliateCodeList = _configServices.GetAffiliateCodeList(request.affiliateId);
    GetCaseHierarchyRequest affiliateRequest = new()
    {
        bigDataAffiliateIdList = $"{string.Join(",", getAffiliateCodeList.Select(n => $"'{n}'"))}"
    };
    if (affiliateRequest.bigDataAffiliateIdList == null) return Unauthorized();
    var caseHierarchyResult = await _currentServices.GetAffiliatesDistributionAsyncNew(request, affiliateRequest.bigDataAffiliateIdList, null);
    if (caseHierarchyResult == null) return NoContent();
    else return Ok(caseHierarchyResult);
}
public class GetAffiliatePlantDistributionRequest
{
    public string? page { get; set; }       
    public string? startDate { get; set; }
    public string? endDate { get; set; }
    public string? kpiCode { get; set; }
    public string? subCategory { get; set; }
    public int? affiliateId { get; set; }
}
List<int> GetAffiliateCodeList(int? affiliateId);
Task<List<GroupedData>> GetAffiliatesDistributionAsyncNew(GetAffiliatePlantDistributionRequest request,string affiliateRequest,int? plantId);
