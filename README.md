public async Task<IActionResult> GetPlantTrend(PlantTrendRequest request)
{
    var result = await _benchMarkServices.GetPlantTrend(request);
    if (result == null) return NoContent();
    else return Ok(result);
}
public class PlantTrendRequest
{
    public string? plantId { get; set; }
    public int? assetClassId { get; set; }
    public int kpiCode { get; set; }
    public string startDate { get; set; } = string.Empty;
    public string endDate { get; set; } = string.Empty;

}
public class PlantTrendResponse
{
    public string? plantName { get; set; }
    public int? plantId { get; set; }        
    public decimal actual { get; set; }
    public decimal absolute { get; set; }
    public string? time { get; set; }
    public string? frequency { get; set; }


}
Task<List<PlantTrendResponse>> GetPlantTrend(PlantTrendRequest request);
