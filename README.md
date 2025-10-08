public async Task<IActionResult> GetCriticalAssetDetails(GetHierarchyInfoRequest request)
{
    var getAffiliateCodeList = _configServices.GetAffiliateCodeList(request.affiliateId);
    GetCaseHierarchyRequest affiliateRequest = new()
    {
        bigDataAffiliateIdList = $"{string.Join(",", getAffiliateCodeList.Select(n => $"'{n}'"))}"
    };
    if (affiliateRequest.bigDataAffiliateIdList == null) return Unauthorized();
    var caseHierarchyResult = await _configServices.GetCriticalAssetDetailsAsyncNew(request.page!, affiliateRequest.bigDataAffiliateIdList);
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
public class GetCriticalAssetDetailsResponse
{
    public string? category { get; set; }
    public string? absolute { get; set; }
    public double? percentage { get; set; }

}
Task<List<GetCriticalAssetDetailsResponse>> GetCriticalAssetDetailsAsyncNew(string page, string bigDataAffiliateRequest);
