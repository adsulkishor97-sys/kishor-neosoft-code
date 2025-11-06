[ApiController]
[Route("api/[controller]")]
public class BenchmarkLoadTestController : ControllerBase
{
    private readonly IBenchMarkServices _benchMarkServices;
    private readonly ILogger<BenchmarkLoadTestController> _logger;

    public BenchmarkLoadTestController(IBenchMarkServices benchMarkServices, ILogger<BenchmarkLoadTestController> logger)
    {
        _benchMarkServices = benchMarkServices;
        _logger = logger;
    }

    // ✅ 1️⃣ Load test for GetAffiliateCriteria
    [HttpPost("LoadTestAffiliateCriteria")]
    public async Task<IActionResult> LoadTestAffiliateCriteria([FromBody] LoadTestCriteriaRequest request)
    {
        if (request == null || request.NoOfRequests <= 0)
            return BadRequest("Invalid request or NoOfRequests must be > 0");

        var stopwatch = System.Diagnostics.Stopwatch.StartNew();
        var successCount = 0;
        var failureCount = 0;
        var failures = new List<string>();

        var tasks = Enumerable.Range(0, request.NoOfRequests).Select(async i =>
        {
            try
            {
                var result = await _benchMarkServices.GetAffiliateCriteria(new GetBenchmarkCriteriaListRequest
                {
                    page = request.Page
                });

                if (result == null)
                {
                    Interlocked.Increment(ref failureCount);
                    lock (failures)
                        failures.Add($"Request #{i + 1} failed: NoContent (Criteria)");
                }
                else
                {
                    Interlocked.Increment(ref successCount);
                }
            }
            catch (Exception ex)
            {
                Interlocked.Increment(ref failureCount);
                lock (failures)
                    failures.Add($"Request #{i + 1} failed with exception: {ex.Message}");
                _logger.LogWarning(ex, "AffiliateCriteria request #{RequestNo} failed", i + 1);
            }
        });

        await Task.WhenAll(tasks);
        stopwatch.Stop();

        return Ok(new
        {
            API = "GetAffiliateCriteria",
            request.NoOfRequests,
            SuccessCount = successCount,
            FailureCount = failureCount,
            Failures = failures,
            TotalTimeMs = stopwatch.ElapsedMilliseconds,
            TotalTimeMinutes = Math.Round(stopwatch.ElapsedMilliseconds / 60000.0, 3),
            AvgTimePerRequestMs = Math.Round(stopwatch.ElapsedMilliseconds / request.NoOfRequests, 2),
            ExecutedAt = DateTime.Now.ToString("yyyy-MM-dd HH:mm:ss")
        });
    }


    // ✅ 2️⃣ Load test for GetAffiliateBenchmarkComparision
    [HttpPost("LoadTestAffiliateComparision")]
    public async Task<IActionResult> LoadTestAffiliateComparision([FromBody] LoadTestComparisionRequest request)
    {
        if (request == null || request.NoOfRequests <= 0)
            return BadRequest("Invalid request or NoOfRequests must be > 0");

        var stopwatch = System.Diagnostics.Stopwatch.StartNew();
        var successCount = 0;
        var failureCount = 0;
        var failures = new List<string>();

        var tasks = Enumerable.Range(0, request.NoOfRequests).Select(async i =>
        {
            try
            {
                var result = await _benchMarkServices.GetAffiliateBenchmarkComparisionAsync(new AffiliateBenchmarkComparisionRequest
                {
                    affiliateId = request.AffiliateId,
                    kpiCode = request.KpiCode,
                    startDate = request.StartDate,
                    endDate = request.EndDate
                });

                if (result == null)
                {
                    Interlocked.Increment(ref failureCount);
                    lock (failures)
                        failures.Add($"Request #{i + 1} failed: NoContent (Comparision)");
                }
                else
                {
                    Interlocked.Increment(ref successCount);
                }
            }
            catch (Exception ex)
            {
                Interlocked.Increment(ref failureCount);
                lock (failures)
                    failures.Add($"Request #{i + 1} failed with exception: {ex.Message}");
                _logger.LogWarning(ex, "AffiliateBenchmarkComparision request #{RequestNo} failed", i + 1);
            }
        });

        await Task.WhenAll(tasks);
        stopwatch.Stop();

        return Ok(new
        {
            API = "GetAffiliateBenchmarkComparision",
            request.NoOfRequests,
            SuccessCount = successCount,
            FailureCount = failureCount,
            Failures = failures,
            TotalTimeMs = stopwatch.ElapsedMilliseconds,
            TotalTimeMinutes = Math.Round(stopwatch.ElapsedMilliseconds / 60000.0, 3),
            AvgTimePerRequestMs = Math.Round(stopwatch.ElapsedMilliseconds / request.NoOfRequests, 2),
            ExecutedAt = DateTime.Now.ToString("yyyy-MM-dd HH:mm:ss")
        });
    }
}


// ✅ Request Models
public class LoadTestCriteriaRequest
{
    public string? Page { get; set; }
    public int NoOfRequests { get; set; }
}

public class LoadTestComparisionRequest
{
    public string? AffiliateId { get; set; }
    public string? KpiCode { get; set; }
    public string? StartDate { get; set; }
    public string? EndDate { get; set; }
    public int NoOfRequests { get; set; }
}
