public async Task<IActionResult> GetPlantBenchmark(GetPlantBenchmarkRequest request)
{

    var caseHierarchyResult = await _benchMarkServices.GetPlantBenchmark(request);
    if (caseHierarchyResult == null) return NoContent();
    else return Ok(caseHierarchyResult);
}
public class GetPlantBenchmarkRequest
{
    public string? plantId { get; set; }
    public string? kpiCode { get; set; }
    public string? startDate { get; set; }
    public string? endDate { get; set; }
    public int assetClassId { get; set; }
}
public class GetPlantBenchmarkResponse
{
    public List<PlantBenchmarkGroupedData> plantgroupedBenchmarkData { get; set; } = new List<PlantBenchmarkGroupedData>();

}

public class PlantBenchmarkGroupedData
{
  
    public int plantId { get; set; }
    public long? target { get; set; }
  
    public int? direction { get; set; }
    public decimal actual { get; set; }
    public decimal absolute { get; set; }
    public decimal bestAchievedEver { get; set; } = 0;
    public decimal bestAchievedEverMin { get; set; } = 0;
    public decimal bestAchievedForSinglePeriod { get; set; } = 0;

}
Task<List<PlantBenchmarkGroupedData>> GetPlantBenchmark(GetPlantBenchmarkRequest request);
