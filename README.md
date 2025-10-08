public async Task<IActionResult> GetPlantList()
{
    var PlantListResult = await _benchMarkServices.GetPlantList();
    if (PlantListResult == null) return NoContent();
    else return Ok(PlantListResult);
}
public class PlantListResponse
{
    public int plantId { get; set; }
    public string? plantName { get; set; }
    public int affiliateId { get; set; }
    public int? productId { get; set; }
    public string? productName { get; set; }
    public string? affiliateName { get; set; }
}
Task<List<PlantListResponse>> GetPlantList();
