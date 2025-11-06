[HttpPost("LoadTestBoth")]
public async Task<IActionResult> LoadTestBoth([FromBody] LoadTestBothRequest request)
{
    if (request == null || request.NoOfRequests <= 0)
        return BadRequest("Invalid request or NoOfRequests must be > 0");

    var stopwatch = System.Diagnostics.Stopwatch.StartNew();

    int failedAffiliateCriteria = 0;
    int failedBenchmarkComparison = 0;
    int successAffiliateCriteria = 0;
    int successBenchmarkComparison = 0;

    var errorLogs = new List<string>();
    var tasks = new List<Task>();

    for (int i = 0; i < request.NoOfRequests; i++)
    {
        // --- Test GetAffiliateCriteria ---
        tasks.Add(Task.Run(async () =>
        {
            try
            {
                var result = await GetAffiliateCriteria(new GetBenchmarkCriteriaListRequest
                {
                    page = request.Page
                });

                if (result is OkObjectResult)
                {
                    Interlocked.Increment(ref successAffiliateCriteria);
                }
                else
                {
                    Interlocked.Increment(ref failedAffiliateCriteria);
                    lock (errorLogs)
                    {
                        errorLogs.Add($"[Criteria] Request #{i + 1} returned non-OK: {result?.GetType().Name}");
                    }
                }
            }
            catch (Exception ex)
            {
                Interlocked.Increment(ref failedAffiliateCriteria);
                lock (errorLogs)
                {
                    errorLogs.Add($"[Criteria] Request #{i + 1} failed: {ex.Message}");
                }
            }
        }));

        // --- Test GetAffiliateBenchmarkComparision ---
        tasks.Add(Task.Run(async () =>
        {
            try
            {
                var result = await GetAffiliateBenchmarkComparision(new AffiliateBenchmarkComparisionRequest
                {
                    affiliateId = request.AffiliateId,
                    kpiCode = request.KpiCode,
                    startDate = request.StartDate,
                    endDate = request.EndDate
                });

                if (result is OkObjectResult)
                {
                    Interlocked.Increment(ref successBenchmarkComparison);
                }
                else
                {
                    Interlocked.Increment(ref failedBenchmarkComparison);
                    lock (errorLogs)
                    {
                        errorLogs.Add($"[Benchmark] Request #{i + 1} returned non-OK: {result?.GetType().Name}");
                    }
                }
            }
            catch (Exception ex)
            {
                Interlocked.Increment(ref failedBenchmarkComparison);
                lock (errorLogs)
                {
                    errorLogs.Add($"[Benchmark] Request #{i + 1} failed: {ex.Message}");
                }
            }
        }));
    }

    await Task.WhenAll(tasks);
    stopwatch.Stop();

    var totalRequests = request.NoOfRequests * 2;
    var totalFailures = failedAffiliateCriteria + failedBenchmarkComparison;
    var totalSuccess = successAffiliateCriteria + successBenchmarkComparison;

    var warningMessage = totalFailures > 0
        ? $"⚠️ WARNING: {totalFailures} requests failed — {failedAffiliateCriteria} (AffiliateCriteria), {failedBenchmarkComparison} (BenchmarkComparision)"
        : "✅ All requests executed successfully.";

    return Ok(new
    {
        TotalRequests = totalRequests,
        SuccessfulRequests = totalSuccess,
        FailedRequests = totalFailures,
        FailedAffiliateCriteria = failedAffiliateCriteria,
        FailedBenchmarkComparision = failedBenchmarkComparison,
        Warning = warningMessage,
        TotalTimeMs = stopwatch.ElapsedMilliseconds,
        TotalTimeMinutes = Math.Round(stopwatch.ElapsedMilliseconds / 60000.0, 3),
        AvgTimePerRequestMs = Math.Round((double)stopwatch.ElapsedMilliseconds / totalRequests, 3),
        StartDate = request.StartDate,
        EndDate = request.EndDate,
        ExecutedAt = DateTime.Now.ToString("yyyy-MM-dd HH:mm:ss"),
        ErrorLogs = errorLogs
    });
}
