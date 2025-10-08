public async Task<IActionResult> GetAsseFilters()
{
    var result = await _benchMarkServices.GetAssetFilters();
    if (result == null) return NoContent();
    else return Ok(result);
}
public class GetAssetFiltersResponse
{
    public string? filterName { get; set; }

    public List<AffiliateList>? affiliateLists { get; set; }

    public List<PlantList>? plantLists { get; set; }

    public List<ProductList>? productLists { get; set; }

    public List<AssetClassLists>? assetClassLists { get; set; }


    public List<AssetManufacturerLists>? assetManufacturerLists { get; set; }

    public List<ModelNumberLists>? modelNumberLists { get; set; }

    public List<ProcessServicesLists>? ProcessServicesLists { get; set; }


}
Task<List<GetAssetFiltersResponse>> GetAssetFilters();
