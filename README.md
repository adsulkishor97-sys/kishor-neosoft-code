public async Task<IActionResult> GetAssetClass()
{
    var AssetClassListResult = await _benchMarkServices.GetAssetClass();
    if (AssetClassListResult == null) return NoContent();
    else return Ok(AssetClassListResult);
}
public class AssetClassResponse
{
    public int assetClassId { get; set; }
    public string? assetClassName { get; set; }
}
Task<List<AssetClassResponse>> GetAssetClass();
