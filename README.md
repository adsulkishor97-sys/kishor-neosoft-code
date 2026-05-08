(List<KpiDataSpResponse>, List<FailureDbResponse>) response =
    await _tocRepository.GetAffiliateKpiBreakdownFailureTrendRolling(request);

var kpiData = response.Item1;
var failureData = response.Item2;

var result = new AssetFailureResponse
{
    subCategory = kpiData.FirstOrDefault()?.subCategory ?? string.Empty,

    failure_methodology = kpiData.FirstOrDefault()?.failureMethodology,

    failures = failureData
        .Select(x => new FailureResponse
        {
            month = x.monthKey,

            count = x.failureCount,

            timeEpoch = x.timeEpoch
        })
        .OrderBy(x => x.month)
        .ToList(),

    kpis = kpiData
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

                    data = periodGroup
                        .OrderBy(x => x.monthKey)
                        .Select(x => new KpiDataResponse
                        {
                            month = x.monthKey,

                            value = Math.Round(x.overallActual, 2),

                            timeEpoch = x.timeEpoch
                        })
                        .ToList()
                })
                .OrderBy(x => x.period_months)
                .ToList()
        })
        .ToList()
};
