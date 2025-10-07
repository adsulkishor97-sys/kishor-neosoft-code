public async Task<IActionResult> GetAssetDistribution(GetAssetDistributionRequest request)
{
    var getAffiliateCodeList = _configServices.GetAffiliateCodeList(request.affiliateId);
    GetCaseHierarchyRequest affiliateRequest = new()
    {
        bigDataAffiliateIdList = $"{string.Join(",", getAffiliateCodeList.Select(n => $"'{n}'"))}"
    };
    if (affiliateRequest.bigDataAffiliateIdList == null) return Unauthorized();
    var caseHierarchyResult = await _currentServices.GetAssetDistributionAsync(request, affiliateRequest.bigDataAffiliateIdList);
    if (caseHierarchyResult == null) return NoContent();
    else return Ok(caseHierarchyResult);
}
public class GetAssetDistributionRequest
{
    public string? page { get; set; }
    public string? startDate { get; set; }
    public string? endDate { get; set; }
    public string? kpiCode { get; set; }
    public string? subCategory { get; set; }
    public int? affiliateId { get; set; }
    public int? plantId { get; set; }
    public string category { get; set; } = string.Empty;
}
public class AssetGroupedData
{
    public string? affiliateId { get; set; }
    public string? name { get; set; }
    public decimal overall { get; set; }
    public decimal critical { get; set; }
    public int overallState { get; set; } = 0;
    public int criticalState { get; set; } = 0;
    public long? overallTarget { get; set; } = null;
    public string? plantId { get; set; }
    public string? plantName { get; set; }
    public string? sapId { get; set; }
    public string? template { get; set; }
    public string? tidnr { get; set; }
    public string? pltxt { get; set; }

}
Task<List<AssetGroupedData>> GetAssetDistributionAsync(GetAssetDistributionRequest request, string affiliateRequest);
