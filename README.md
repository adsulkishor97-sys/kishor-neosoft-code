[HttpPost("LoadTestBoth")]
public async Task<IActionResult> LoadTestBoth([FromBody] LoadTestBothRequest request)
{
    if (request == null || request.NoOfRequests <= 0)
        return BadRequest("Invalid request or NoOfRequests must be > 0");

    var stopwatch = System.Diagnostics.Stopwatch.StartNew();

    // Run both APIs in parallel for N requests
    var tasks = new List<Task>();

    for (int i = 0; i < request.NoOfRequests; i++)
    {
        // Run Affiliate Criteria
        tasks.Add(Task.Run(async () =>
        {
            await GetAffiliateCriteria(new GetBenchmarkCriteriaListRequest
            {
                page = request.Page
            });
        }));

        // Run Affiliate Benchmark Comparison
        tasks.Add(Task.Run(async () =>
        {
            await GetAffiliateBenchmarkComparision(new AffiliateBenchmarkComparisionRequest
            {
                affiliateId = request.AffiliateId,
                kpiCode = request.KpiCode,
                startDate = request.StartDate,
                endDate = request.EndDate
            });
        }));
    }

    await Task.WhenAll(tasks);
    stopwatch.Stop();

    // Response summary
    return Ok(new
    {
        TotalRequestsExecuted = request.NoOfRequests * 2,
        TotalTimeMs = stopwatch.ElapsedMilliseconds,
        AvgTimePerRequestMs = stopwatch.ElapsedMilliseconds / (request.NoOfRequests * 2),
        StartDate = request.StartDate,
        EndDate = request.EndDate,
        ExecutedAt = DateTime.Now.ToString("yyyy-MM-dd HH:mm:ss")
    });
}
public class LoadTestBothRequest
{
    public int NoOfRequests { get; set; } = 10;
    public string? Page { get; set; }
    public string? AffiliateId { get; set; }
    public string? KpiCode { get; set; }
    public string? StartDate { get; set; }
    public string? EndDate { get; set; }
}
