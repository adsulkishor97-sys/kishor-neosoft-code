public async Task<IActionResult> GetPlantFilters()
{
    var PlantListResult = await _benchMarkServices.GetPlantFilters();
    if (PlantListResult == null) return NoContent();
    else return Ok(PlantListResult);
}
public class FinalFilterResponse
{
    public string? filterName { get; set; }
    public List<AffiliateFiltersResponse>? affiliateLists { get; set; }
    public List<ProductList>? productLists { get; set; }
}
Task<List<FinalFilterResponse>> GetPlantFilters();
