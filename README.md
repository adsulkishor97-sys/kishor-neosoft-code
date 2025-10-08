 public async Task<IActionResult> GetCaseHierarchy()
 {
     var getAffiliateIdList = _configServices.GetAffiliateIdList(null);
     GetCaseHierarchyRequest request = new()
     {
         affiliateIdList = $"{string.Join(",", getAffiliateIdList.Select(n => $"{n}"))}"
     };
     var caseHierarchyResult = await _configServices.GetCaseHierarchyAsync(request);
     if (caseHierarchyResult == null) return NoContent();
     else return Ok(caseHierarchyResult);

 }
 public class GetCaseHierarchyRequest
{
    public int? affiliateId { get; set; }
    public int? plantId { get; set; }
    public string? bigDataAffiliateIdList {  get; set; }
    public string? affiliateIdList { get; set; }
}
public class CaseHierarchyResponse
{
    public string? regionName { get; set; }
    public string? regionShortName { get; set; }
    public List<AffiliateHierarchy>? affiliates { get; set; }
}
public class AffiliateHierarchy
{
    public string? affiliateName { get; set; }
    public string? affiliateImage { get; set; }
    public int? affiliateID { get; set; }
    public string? isSeec { get; set; }
    public List<PlantHierarchy>? plants { get; set; }
}
 Task<List<CaseHierarchyResponse>> GetCaseHierarchyAsync(GetCaseHierarchyRequest request);
