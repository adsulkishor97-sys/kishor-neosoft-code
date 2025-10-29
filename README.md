public async Task<List<AssetsPerformanceKpiTrendsResponse>> GetMonthlyAndQuarterlyAsync(string piTag, DateTime stime, DateTime etime)
{
    var results = new List<AssetsPerformanceKpiTrendsResponse>();

    foreach (var server in _serverHelper.ConnectedPIServers)
    {
        PIPoint piPoint = null;
        try
        {
            piPoint = _pIPointWrapper.FindPIPoint(server, piTag);
                
        }
        catch
        {
            continue;
        }
        if (piPoint != null)
        {
            results.AddRange(await GetSummariesAsync(piPoint, stime, etime, "month", 1));
            results.AddRange(await GetSummariesAsync(piPoint, stime, etime, "quarter", 3));
            break;
        }
    }
    return results;
}
       
private async Task<IEnumerable<AssetsPerformanceKpiTrendsResponse>> GetSummariesAsync(PIPoint piPoint, DateTime stime, DateTime etime, string period, int stepMonths)
{
    var summaries = new List<AssetsPerformanceKpiTrendsResponse>();
    DateTime cursor = AlignToPeriodStart(stime, stepMonths);

    while (cursor < etime)
    {
        DateTime periodStart = (cursor < stime) ? stime : cursor;
        DateTime periodEnd = cursor.AddMonths(stepMonths);
        if (periodEnd > etime) periodEnd = etime;
        if (periodStart >= periodEnd) break;

        AFTimeRange afTimeRange = new AFTimeRange(periodStart, periodEnd);

        var summary = await piPoint.SummaryAsync(
            afTimeRange,
            AFSummaryTypes.Average,
            AFCalculationBasis.TimeWeighted,
            AFTimestampCalculation.Auto
        );

        double? val = null;
        try
        {

            val = summary[AFSummaryTypes.Average].ValueAsDouble();
        }
        catch
        {
            // skip exception to set the value as null
        }
        long epoch = new DateTimeOffset(cursor).ToUnixTimeMilliseconds();

        summaries.Add(new AssetsPerformanceKpiTrendsResponse
        {
            tagName = piPoint.Name,
            time = epoch.ToString(),
            value = val,
            frequency = period,
            label = BuildLabel(cursor, stepMonths)
        });

        cursor = cursor.AddMonths(stepMonths);
    }

    return summaries;
}
