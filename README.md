using System;
using System.Collections.Generic;
using System.Data.Common;
using System.Globalization;
using System.Linq;
using System.Threading.Tasks;

public class AffiliateBenchmarkContext
{
    public GetAffiliateBenchmarkRequest Request { get; set; } = default!;
    public string FormattedStartDate { get; set; } = string.Empty;
    public string FormattedEndDate { get; set; } = string.Empty;
    public string FormattedStartDateYMD { get; set; } = string.Empty;
    public string FormattedEndDateYMD { get; set; } = string.Empty;
    public List<KpiDataJsonResponse> SqlKpiDataList { get; set; } = new();
    public List<AffiliateListResponse> SqlAffiliateName { get; set; } = new();
    public KpiDataJsonResponse? KpiInfo { get; set; }
}

public class AffiliateBenchmarkQueryParams
{
    public AffiliateBenchmark Item { get; set; } = default!;
    public AffiliateBenchmarkContext Context { get; set; } = default!;
}

public class AffiliateBenchmarkService
{
    private readonly IBenchMarkRepository _benchMarkRepository;

    public AffiliateBenchmarkService(IBenchMarkRepository benchMarkRepository)
    {
        _benchMarkRepository = benchMarkRepository;
    }

    public async Task<List<BenchmarkGroupedData>> GetAffiliateBenchmark(GetAffiliateBenchmarkRequest request)
    {
        // 1️⃣ Format Dates
        var (formattedStartdate, formattedEnddate, formattedStartdateYMD, formattedEnddateYMD) = FormatDatesAffiliate(request);

        // 2️⃣ Get KPI data
        var sqlKpiDataList = await _benchMarkRepository.GetBusinessKPIDetails();
        var kpiInfo = sqlKpiDataList.FirstOrDefault(x => x.kpiCode == request.kpiCode);

        // 3️⃣ Get Affiliate Name list
        var sqlAffiliateName = (await _benchMarkRepository.GetAffiliateLists())
            .Select(a => new AffiliateListResponse
            {
                affiliateId = a.affiliateId,
                affiliateName = a.affiliateName
            }).ToList();

        // 4️⃣ Build context
        var context = new AffiliateBenchmarkContext
        {
            Request = request,
            FormattedStartDate = formattedStartdate,
            FormattedEndDate = formattedEnddate,
            FormattedStartDateYMD = formattedStartdateYMD,
            FormattedEndDateYMD = formattedEnddateYMD,
            SqlKpiDataList = sqlKpiDataList,
            SqlAffiliateName = sqlAffiliateName,
            KpiInfo = kpiInfo
        };

        // 5️⃣ Process the request
        return await ProcessAffiliateBenchmarkAsync(context);
    }

    private static (string, string, string, string) FormatDatesAffiliate(GetAffiliateBenchmarkRequest request)
    {
        var startDateTime = DateTime.Parse(request.startDate!, CultureInfo.InvariantCulture);
        var endDateTime = DateTime.Parse(request.endDate!, CultureInfo.InvariantCulture);

        var formattedStartdate = startDateTime.ToString("yyyyMM");
        var formattedEnddate = endDateTime.ToString("yyyyMM");
        var formattedStartdateYMD = startDateTime.ToString("yyyy-MM-dd");
        var formattedEnddateYMD = endDateTime.ToString("yyyy-MM-dd");

        return (formattedStartdate, formattedEnddate, formattedStartdateYMD, formattedEnddateYMD);
    }

    private async Task<List<BenchmarkGroupedData>> ProcessAffiliateBenchmarkAsync(AffiliateBenchmarkContext context)
    {
        var affiliateBenchmarkQuery = new AffiliateBenchmarkQuery();
        var responseList = affiliateBenchmarkQuery.affiliateDataList
            .Where(a => a.kpiCode == context.Request.kpiCode)
            .ToList();

        List<BenchmarkGroupedData> groupData = new();

        foreach (var item in responseList)
        {
            var queryParams = new AffiliateBenchmarkQueryParams
            {
                Item = item,
                Context = context
            };

            var result = await ExecuteAffiliateBenchmarkQueriesAsync(queryParams);
            groupData.AddRange(result);
        }

        return groupData;
    }

    private async Task<List<BenchmarkGroupedData>> ExecuteAffiliateBenchmarkQueriesAsync(AffiliateBenchmarkQueryParams queryParams)
    {
        var item = queryParams.Item;
        var ctx = queryParams.Context;

        var actualQueryStr = AffilateReplaceQuery(
            item.query!,
            ctx.FormattedStartDateYMD, ctx.FormattedEndDateYMD,
            ctx.FormattedStartDate, ctx.FormattedEndDate,
            ctx.Request
        );

        var bestQueryStr = AffilateReplaceQuery(
            item.bestAchievedEver!,
            ctx.FormattedStartDateYMD, ctx.FormattedEndDateYMD,
            ctx.FormattedStartDate, ctx.FormattedEndDate,
            ctx.Request
        );

        var actualQuery = await ExecuteAffiliateDataQueryAsync(actualQueryStr, ctx.SqlAffiliateName);
        var bestQuery = await ExecuteAffiliateDataQueryAsync(bestQueryStr, ctx.SqlAffiliateName, isBest: true);

        if (!actualQuery.Any() || string.IsNullOrEmpty(bestQueryStr))
            return new List<BenchmarkGroupedData>();

        var mergedData = MergeAffiliateBenchmarkData(actualQuery, bestQuery);

        if (!string.IsNullOrEmpty(ctx.Request.kpiCode))
        {
            UpdateComputedFields(mergedData, ctx.SqlKpiDataList, ctx.KpiInfo, ctx.Request.kpiCode);
        }

        return mergedData;
    }

    private async Task<List<BenchmarkGroupedData>> ExecuteAffiliateDataQueryAsync(
        string query,
        List<AffiliateListResponse> sqlAffiliateName,
        bool isBest = false)
    {
        return await _benchMarkRepository.ExecuteBigDataQuery<BenchmarkGroupedData>(
            query,
            reader =>
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
            bestAchievedEverMin = groupData.Any() ? groupData.Min(x => x.bestAchievedEver) : 0;
            bestAchievedForSinglePeriod = groupData.Any() ? groupData.Min(x => x.actual) : 0;
        }
        else
        {
            bestAchievedEverMin = groupData.Any() ? groupData.Max(x => x.bestAchievedEver) : 0;
            bestAchievedForSinglePeriod = groupData.Any() ? groupData.Max(x => x.actual) : 0;
        }

        var sqlDirection = sqlKpiDataList.FirstOrDefault(x => x.kpiCode == kpiCode)?.direction;
        var targetMin = sqlKpiDataList.FirstOrDefault(x => x.kpiCode == kpiCode)?.overallTargetMin;
        var targetMax = sqlKpiDataList.FirstOrDefault(x => x.kpiCode == kpiCode)?.overallTargetMax;

        foreach (var item in groupData)
        {
            item.state = (item.actual >= targetMin && item.actual <= targetMax) ? 1 : 0;
            item.direction = sqlDirection;
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
