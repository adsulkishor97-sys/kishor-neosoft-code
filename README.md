public async Task<IActionResult> GetPlantBenchmarkComparision(PlantBenchmarkComparisionRequest plantBenchmarkComparisionRequest)
{
    var plantComparisionResult = await _benchMarkServices.GetPlantBenchmarkComparisionAsync(plantBenchmarkComparisionRequest);
    if (plantComparisionResult == null) return NoContent();
    else return Ok(plantComparisionResult);
}
public class PlantBenchmarkComparisionRequest
{
    public string? plantId { get; set; }
    public string? kpiCode { get; set; }
    public string? startDate { get; set; }
    public string? endDate { get; set; }
    public int? assetClassId { get; set; }
}
public class PlantBenchmarkComparisionResponse
{
    public int? plantId { get; set; }
    public decimal? actual { get; set; }
    public decimal? absolute { get; set; }
}
Task<List<PlantBenchmarkComparisionResponse>> GetPlantBenchmarkComparisionAsync(PlantBenchmarkComparisionRequest plantBenchmarkComparisionRequest);
