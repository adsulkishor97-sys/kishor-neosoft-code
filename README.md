public async Task<IActionResult> GetAsseList(GetAssetListRequest getAssetListRequest)
{
    var result = await _benchMarkServices.GetAssetList(getAssetListRequest);
    if (result == null) return NoContent();
    else return Ok(result);
}
public class GetAssetListRequest
{

    public string? affiliateId { get; set; } 
    public string? productId { get; set; } 
    public string? plantId { get; set; } 
    public string? processServiceName { get; set; } 
    public string? assetClassId { get; set; } 
    public string? designClassificationName { get; set; } 
    public string? modelNumber { get; set; } 
    public string? companyName { get; set; } 


}
public class GetAssetListResponse
{
    public string? assetId { get; set; }
    public string? sapId { get; set; }
    public string? assetName { get; set; }
    public int affiliateId { get; set; }
    public string? affiliateName { get; set; }
    public int productId { get; set; }
    public int plantId { get; set; }
    public string? plantName { get; set; }

    public string? processServiceName { get; set; }
    public string? modelNumber { get; set; }
    public int assetClassId { get; set; }
    public string? assetClassName { get; set; }

    public string? designClassificationName { get; set; }
    public string? companyName { get; set; }

}
Task<List<GetAssetListResponse>> GetAssetList(GetAssetListRequest getAssetListRequest);
