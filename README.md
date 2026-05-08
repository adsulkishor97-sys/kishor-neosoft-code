public class KpiDbResponse
{
    public int rollingPeriod { get; set; }
    public int kpiCode { get; set; }
    public string kpiName { get; set; }
    public string subCategory { get; set; }
    public string? failureMethodology { get; set; }
    public string monthKey { get; set; }
    public double overallActual { get; set; }
    public int direction { get; set; }
    public long timeEpoch { get; set; }
}
public class FailureDbResponse
{
    public string monthKey { get; set; }
    public int failureCount { get; set; }
}
var groupedKpis = sqlData
    .GroupBy(x => new
    {
        x.kpiCode,
        x.kpiName,
        x.direction
    })
    .Select(kpiGroup => new KpiResponse
    {
        kpi_code = kpiGroup.Key.kpiCode.ToString(),
        kpi_name = kpiGroup.Key.kpiName,
        direction = kpiGroup.Key.direction,

        rolling_averages = kpiGroup
            .GroupBy(x => x.rollingPeriod)
            .Select(periodGroup => new RollingAverageResponse
            {
                period_months = periodGroup.Key,

                data = periodGroup.Select(x => new KpiDataResponse
                {
                    month = x.monthKey,
                    value = x.overallActual,
                    timeEpoch = x.timeEpoch
                }).ToList()
            }).ToList()
    }).ToList();
