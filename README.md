public async Task<IActionResult> GetHierarchyInfo(GetHierarchyInfoRequest request)
{
    var getAffiliateCodeList = _configServices.GetAffiliateCodeList(request.affiliateId);
    GetCaseHierarchyRequest affiliateRequest = new()
    {
        affiliateIdList = $"{string.Join(",", getAffiliateCodeList.Select(n => $"{n}"))}",
        bigDataAffiliateIdList = $"{string.Join(",", getAffiliateCodeList.Select(n => $"'{n}'"))}"
    };
    if (affiliateRequest.affiliateIdList == null) return Unauthorized();
    var caseHierarchyResult = await _configServices.GetHierarchyInfoAsync(request.page!, affiliateRequest.affiliateIdList, affiliateRequest.bigDataAffiliateIdList, request.affiliateId!, request.plantId);
    if (caseHierarchyResult == null) return NoContent();

    else return Ok(caseHierarchyResult);

}
 public class GetHierarchyInfoRequest
 {
     public int? affiliateId { get; set; }
     public int? plantId { get; set; }
     public string? page { get; set; }
 }
  public class GetCaseHierarchyRequest
 {
     public int? affiliateId { get; set; }
     public int? plantId { get; set; }
     public string? bigDataAffiliateIdList {  get; set; }
     public string? affiliateIdList { get; set; }
 }
 public class GetHierarchyInfoResponse
{
    public int? affiliates { get; set; }
    public int? plants { get; set; }
    public int? product { get; set; }
    public int? assetClass { get; set; }
    public int? assets { get; set; }
    public int?  criticalAssets { get; set; }
}
Task<GetHierarchyInfoResponse?> GetHierarchyInfoAsync(string page, string sqlAffiliateRequest, string bigDataAffiliateRequest, int? affiliateId, int? plantId);
