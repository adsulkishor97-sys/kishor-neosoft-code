using Microsoft.AspNetCore.Mvc;
using System.Diagnostics;

namespace YourNamespace.Controllers
{
    [ApiController]
    [Route("api/[controller]")]
    public class BenchmarkLoadTestController : ControllerBase
    {
        private readonly IBenchmarkRepository _benchmarkRepository; // SQL repo
        private readonly IBenchMarkService _benchmarkService;       // Big Data service

        public BenchmarkLoadTestController(
            IBenchmarkRepository benchmarkRepository,
            IBenchMarkService benchmarkService)
        {
            _benchmarkRepository = benchmarkRepository;
            _benchmarkService = benchmarkService;
        }

        /// <summary>
        /// Run load test with dynamic JSON parameters for SQL & Big Data.
        /// </summary>
        [HttpPost("run")]
        public async Task<IActionResult> RunLoadTest([FromBody] BenchmarkLoadTestRequest request)
        {
            if (request == null)
                return BadRequest("Request body cannot be null");

            var stopwatch = Stopwatch.StartNew();
            var tasks = new List<Task>();

            for (int i = 0; i < request.ConcurrentRequests; i++)
            {
                tasks.Add(RunSqlCallAsync(request.SqlRequest));
                tasks.Add(RunBigDataCallAsync(request.BigDataRequest));
            }

            await Task.WhenAll(tasks);
            stopwatch.Stop();

            return Ok(new
            {
                SqlRequestParameters = request.SqlRequest,
                BigDataRequestParameters = request.BigDataRequest,
                TotalRequestsSent = tasks.Count,
                ConcurrentRequestsPerType = request.ConcurrentRequests,
                TotalElapsedMilliseconds = stopwatch.ElapsedMilliseconds,
                AveragePerRequestMs = stopwatch.ElapsedMilliseconds / (double)tasks.Count,
                Message = "Load test completed successfully"
            });
        }

        private async Task RunSqlCallAsync(GetBenchmarkCriteriaListRequest sqlRequest)
        {
            try
            {
                await _benchmarkRepository.GetAffiliateCriteria(sqlRequest);
            }
            catch
            {
                // log or ignore for load test
            }
        }

        private async Task RunBigDataCallAsync(AffiliateBenchmarkComparisionRequest bigDataRequest)
        {
            try
            {
                await _benchmarkService.GetAffiliateBenchmarkComparisionAsync(bigDataRequest);
            }
            catch
            {
                // log or ignore for load test
            }
        }
    }
}

 public class BenchmarkLoadTestRequest
    {
        /// <summary>
        /// Number of concurrent requests to send (per type: SQL + Big Data)
        /// </summary>
        public int ConcurrentRequests { get; set; } = 10;

        /// <summary>
        /// SQL Server request details
        /// </summary>
        public GetBenchmarkCriteriaListRequest SqlRequest { get; set; } = new();

        /// <summary>
        /// Big Data Server request details
        /// </summary>
        public AffiliateBenchmarkComparisionRequest BigDataRequest { get; set; } = new();
    }
