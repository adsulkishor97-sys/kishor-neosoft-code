public async Task<IActionResult> LoadTestBoth([FromBody] LoadTestBothRequest request)
{
    if (request == null || request.NoOfRequests <= 0)
        return BadRequest("Invalid request or NoOfRequests must be > 0");

    var stopwatch = System.Diagnostics.Stopwatch.StartNew();

    var tasks = new List<Task>();

    for (int i = 0; i < request.NoOfRequests; i++)
    {
        
        tasks.Add(Task.Run(async () =>
        {
            await GetAffiliateCriteria(new GetBenchmarkCriteriaListRequest
            {
                page = request.Page
            });
        }));

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

    return Ok(new
    {
        TotalTimeMs = stopwatch.ElapsedMilliseconds,
        TotalTimeMinutes = Math.Round(stopwatch.ElapsedMilliseconds / 60000.0, 3),
        AvgTimePerRequestMs = stopwatch.ElapsedMilliseconds / (request.NoOfRequests * 2),
        StartDate = request.StartDate,
        EndDate = request.EndDate,
        ExecutedAt = DateTime.Now.ToString("yyyy-MM-dd HH:mm:ss")
    });
}
