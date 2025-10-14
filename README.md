public async Task<IActionResult> GetSearchHistoryBySearchKey(GetSearchHistoryRequest request)
{
    int userId = CommonMethod.GetEmployeeIdFromClaim(HttpContext);
    var assetListResultData = await _searchHistoryServices.GetSearchHistoryBySearchKeyAsync(request, userId);
    if (assetListResultData! == null) return NoContent();
    else return Ok(assetListResultData);
}
public class GetSearchHistoryRequest
{     
    public string? searchKey { get; set; }

}
 public class SearchHistoryResponse
 {
     public int? userId { get; set; }
     public string? searchKey { get; set; }
     public string? searchValue { get; set; }
     public string? createdOn { get; set; }
 }
public static int GetEmployeeIdFromClaim(HttpContext context)
{
    var claims = context.User.Identity as ClaimsIdentity;
    Claim? claimUID = claims!.Claims.FirstOrDefault(claim => claim.Type == "uid");
    int createdBy = Convert.ToInt32(claimUID!.Value);
    return createdBy;
}
Task<List<string>> GetSearchHistoryBySearchKeyAsync(GetSearchHistoryRequest searchHistoryRequest, int userId);
