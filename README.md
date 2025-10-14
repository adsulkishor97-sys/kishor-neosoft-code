public async Task<IActionResult> AddSearchHistoryBySearchKey(SearchHistoryRequest request)
{
    int userId = CommonMethod.GetEmployeeIdFromClaim(HttpContext);
    int? searchHistoryId = await _searchHistoryServices.AddSearchHistoryBySearchKeyAsync(request, userId);
    if (searchHistoryId == null || searchHistoryId <= 0)
    {
        return NoContent();
    }
    else
    {
        return Ok(searchHistoryId);
    }
}
public class SearchHistoryRequest
{
    public string? searchKey { get; set; }
    public string? searchValue { get; set; }
    
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
 Task<int?> AddSearchHistoryBySearchKeyAsync(SearchHistoryRequest searchHistoryRequest, int userId);
