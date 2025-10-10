using System;
using System.Collections.Generic;
using System.Data.Common;
using System.Globalization;
using System.Linq;
using System.Threading.Tasks;

public class AffiliateBenchmarkService
{
    private readonly IBenchMarkRepository _benchMarkRepository;

    public AffiliateBenchmarkService(IBenchMarkRepository benchMarkRepository)
    {
        _benchMarkRepository = benchMarkRepository;
    }

    public async Task<List<BenchmarkGroupedData>> GetAffiliateBenchmark(GetAffiliateBenchmarkRequest request)
    {
        var dateRange = FormatDatesAffiliate(request);

        var sqlKpiDataList = await _benchMarkRepository.GetBusinessKPIDetails();
        var kpiInfo = sqlKpiDataList.Find(x => x.kpiCode == request.kpiCode);

        var sqlAffiliateName = (await _benchMarkRepository.GetAffiliateLists())
            .Select(a => new AffiliateListResponse
            {
                affiliateId = a.affiliateId,
                affiliateName = a.affiliateName
            })
            .ToList();

        var context = new QueryContext(sqlKpiDataList, sqlAffiliateName, kpiInfo);

        return await ProcessAffiliateBenchmarkAsync(request, dateRange, context);
    }

    private static DateRange FormatDatesAffiliate(GetAffiliateBenchmarkRequest request)
    {
        var start = DateTime.Parse(request.startDate!, CultureInfo.InvariantCulture);
        var end = DateTime.Parse(request.endDate!, CultureInfo.InvariantCulture);

        return new DateRange
        {
            StartMonth = start.ToString("yyyyMM"),
            EndMonth = end.ToString("yyyyMM"),
            StartDateYMD = start.ToString("yyyy-MM-dd"),
            EndDateYMD = end.ToString("yyyy-MM-dd")
        };
    }

    private async Task<List<BenchmarkGroupedData>> ProcessAffiliateBenchmarkAsync(
        GetAffiliateBenchmarkRequest request,
        DateRange dateRange,
        QueryContext context)
    {
        var responseList = new AffiliateBenchmarkQuery().affiliateDataList
            .Where(a => a.kpiCode == request.kpiCode)
            .ToList();

        var allData = new List<BenchmarkGroupedData>();

        foreach (var item in responseList)
        {
            var result = await ExecuteAffiliateBenchmarkQueriesAsync(item, request, dateRange, context);
            allData.AddRange(result);
        }

        return allData;
    }

    private async Task<List<BenchmarkGroupedData>> ExecuteAffiliateBenchmarkQueriesAsync(
        AffiliateBenchmark item,
        GetAffiliateBenchmarkRequest request,
        DateRange dateRange,
        QueryContext context)
    {
        var (actualQuery, bestQuery) = BuildAffiliateQueries(item, request, dateRange);

        var actualData = await ExecuteAffiliateDataQueryAsync(actualQuery, context.SqlAffiliateName);
        var bestData = await ExecuteAffiliateDataQueryAsync(bestQuery, context.SqlAffiliateName, isBest: true);

        if (actualData.Count == 0 || string.IsNullOrEmpty(bestQuery))
            return new List<BenchmarkGroupedData>();

        var mergedData = MergeAffiliateBenchmarkData(actualData, bestData);
        if (request?.kpiCode != null)
        {
            UpdateComputedFields(mergedData, context.SqlKpiDataList, context.KpiInfo, request.kpiCode);
        }

        return mergedData;
    }

    private static (string actual, string best) BuildAffiliateQueries(
        AffiliateBenchmark item,
        GetAffiliateBenchmarkRequest request,
        DateRange dateRange)
    {
        string actualQuery = AffilateReplaceQuery(
            item.query!,
            dateRange.StartDateYMD, dateRange.EndDateYMD,
            dateRange.StartMonth, dateRange.EndMonth,
            "",
            request);

        string bestQuery = AffilateReplaceQuery(
            item.bestAchievedEver!,
            dateRange.StartDateYMD, dateRange.EndDateYMD,
            dateRange.StartMonth, dateRange.EndMonth,
            "",
            request);

        return (actualQuery, bestQuery);
    }

    private async Task<List<BenchmarkGroupedData>> ExecuteAffiliateDataQueryAsync(
        string query,
        List<AffiliateListResponse> sqlAffiliateName,
        bool isBest = false)
    {
        if (string.IsNullOrWhiteSpace(query))
            return new List<BenchmarkGroupedData>();

        return await _benchMarkRepository.ExecuteBigDataQuery(query, reader =>
        {
            int affiliateId = GetValueOrDefault(reader, "affiliate_id", 0);
            string? affiliateName = sqlAffiliateName.Find(x => x.affiliateId == affiliateId)?.affiliateName;

            return isBest
                ? new BenchmarkGroupedData
                {
                    affiliateName = affiliateName,
                    bestAchievedEver = GetValueOrDefault(reader, "best_achieved", 0m)
                }
                : new BenchmarkGroupedData
                {
                    affiliateName = affiliateName,
                    actual = GetValueOrDefault(reader, "actual", 0m),
                    absolute = GetValueOrDefault(reader, "absolute", 0m)
                };
        }) ?? new List<BenchmarkGroupedData>();
    }

    private static List<BenchmarkGroupedData> MergeAffiliateBenchmarkData(
        List<BenchmarkGroupedData> actualData,
        List<BenchmarkGroupedData> bestData)
    {
        return (from a in actualData
                join b in bestData on a.affiliateName equals b.affiliateName into temp
                from b in temp.DefaultIfEmpty()
                select new BenchmarkGroupedData
                {
                    affiliateName = a.affiliateName,
                    actual = a.actual,
                    absolute = a.absolute,
                    bestAchievedEver = b?.bestAchievedEver ?? 0
                })
            .GroupBy(x => x.affiliateName)
            .Select(g => new BenchmarkGroupedData
            {
                affiliateName = g.Key,
                actual = g.Sum(x => x.actual),
                absolute = g.Sum(x => x.absolute),
                bestAchievedEver = g.Max(x => x.bestAchievedEver)
            })
            .ToList();
    }

    private static void UpdateComputedFields(
        List<BenchmarkGroupedData> groupData,
        List<KpiDataJsonResponse> sqlKpiDataList,
        KpiDataJsonResponse? kpiInfo,
        string kpiCode)
    {
        decimal bestAchievedEverMin, bestAchievedForSinglePeriod;

        if (kpiInfo?.direction == 1)
        {
            bestAchievedEverMin = groupData.Count > 0 ? groupData.Min(x => x.bestAchievedEver) : 0;
            bestAchievedForSinglePeriod = groupData.Count > 0 ? groupData.Min(x => x.actual) : 0;
        }
        else
        {
            bestAchievedEverMin = groupData.Count > 0 ? groupData.Max(x => x.bestAchievedEver) : 0;
            bestAchievedForSinglePeriod = groupData.Count > 0 ? groupData.Max(x => x.actual) : 0;
        }

        var kpi = sqlKpiDataList.Find(x => x.kpiCode == kpiCode);
        var direction = kpi?.direction;
        var targetMin = kpi?.overallTargetMin;
        var targetMax = kpi?.overallTargetMax;

        foreach (var item in groupData)
        {
            item.state = (item.actual >= targetMin && item.actual <= targetMax) ? 1 : 0;
            item.direction = direction;
            item.target = targetMax ?? 0;
            item.bestAchievedEverMin = bestAchievedEverMin;
            item.bestAchievedForSinglePeriod = bestAchievedForSinglePeriod;
        }
    }

    private static T GetValueOrDefault<T>(DbDataReader reader, string columnName, T defaultValue = default!)
    {
        try
        {
            int ordinal = reader.GetOrdinal(columnName);
            if (reader.IsDBNull(ordinal))
                return defaultValue;

            return (T)Convert.ChangeType(reader.GetValue(ordinal), typeof(T));
        }
        catch
        {
            return defaultValue;
        }
    }
}

#region Helper Models

public record DateRange
{
    public string StartMonth { get; init; } = "";
    public string EndMonth { get; init; } = "";
    public string StartDateYMD { get; init; } = "";
    public string EndDateYMD { get; init; } = "";
}

public class QueryContext
{
    public List<KpiDataJsonResponse> SqlKpiDataList { get; }
    public List<AffiliateListResponse> SqlAffiliateName { get; }
    public KpiDataJsonResponse? KpiInfo { get; }

    public QueryContext(List<KpiDataJsonResponse> kpiList, List<AffiliateListResponse> affiliateList, KpiDataJsonResponse? kpiInfo)
    {
        SqlKpiDataList = kpiList;
        SqlAffiliateName = affiliateList;
        KpiInfo = kpiInfo;
    }
}

#endregion
