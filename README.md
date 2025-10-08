public async Task<IActionResult> GetAssetTrend(AssetTrendRequest request)
{
    var result = await _benchMarkServices.GetAssetTrend(request);
    if (result == null) return NoContent();
    else return Ok(result);
}
public class AssetTrendRequest
{
    public string? sapId { get; set; }
    public int kpiCode { get; set; }
    public string startDate { get; set; } = string.Empty;
    public string endDate { get; set; } = string.Empty;
}
public class AssetTrendResponse
{
    public string? sapId { get; set; }
    public decimal actual { get; set; }
    public decimal absolute { get; set; }
    public string? time { get; set; }
    public string? frequency { get; set; }
}
Task<List<AssetTrendResponse>> GetAssetTrend(AssetTrendRequest request);
