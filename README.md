public async Task<IActionResult> GetBenchmarkDesignBySapId(GetBenchmarkDesignBySapIdRequest request)
{
    var PlantListResult = await _benchMarkServices.GetBenchmarkDesignBySapId(request);
    if (PlantListResult == null) return NoContent();
    else return Ok(PlantListResult);
}
public class GetBenchmarkDesignBySapIdRequest
{
    public string? sapId {  get; set; }
    public string? kpiCode {  get; set; }

}
public class GetBenchmarkDesignBySapIdResponse
{
    public string? sapId {  get; set; }
    public string? target {  get; set; }
    public int direction {  get; set; }
    public string? actualValue { get; set; }
    public string? absolute {  get; set; }
    public string? bestAchievedEver {  get; set; }
    public string? bestAchievedEverMin {  get; set; }
    public string? bestAchievedForSinglePeriod {  get; set; }
    public string? modelNo {  get; set; }
    public string? manufacturer {  get; set; }
    public string? designValue {  get; set; }
}
 Task<List<GetBenchmarkDesignBySapIdResponse>> GetBenchmarkDesignBySapId(GetBenchmarkDesignBySapIdRequest request);
