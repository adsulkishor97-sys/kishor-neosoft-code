public async Task<IActionResult> GetAffiliateList()
{
    var affiliateListResult = await _benchMarkServices.GetAffiliateList();
    if (affiliateListResult == null) return NoContent();
    else return Ok(affiliateListResult);
}
public class AffiliateListResponse
{
    public int affiliateId { get; set; }
    public string? affiliateName { get; set; }
}
Task<List<AffiliateListResponse>> GetAffiliateList();
