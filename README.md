using System;
using System.Collections.Generic;
using System.Globalization;
using System.Linq;
using System.Threading.Tasks;

public class AssetBenchmarkService
{
    private readonly IBenchmarkRepository _benchMarkRepository;

    public AssetBenchmarkService(IBenchmarkRepository benchMarkRepository)
    {
        _benchMarkRepository = benchMarkRepository;
    }

    public async Task<List<AssetBenchmarkGroupedData>> GetAssetBenchmark(GetAssetBenchmarkRequest request)
    {
        if (request == null) return new List<AssetBenchmarkGroupedData>();

        var (formattedStartDate, formattedEndDate, formattedStartDateYMD, formattedEndDateYMD) =
            FormatDates(request.startDate!, request.endDate!);

        var formattedSapIds = FormatSapIds(request.sapId);

        var sqlKpiDataList = await _benchMarkRepository.GetBusinessKPIDetails();
        var kpiInfo = sqlKpiDataList.FirstOrDefault(x => x.kpiCode == request.kpiCode);

        var resultData = await GetAssetBenchmarkDataAsync(
            request,
            formattedStartDate,
            formattedEndDate,
            formattedStartDateYMD,
            formattedEndDateYMD,
            formattedSapIds,
            string.Empty);

        if (resultData == null || resultData.Count == 0)
            return new List<AssetBenchmarkGroupedData>();

        if (!string.IsNullOrEmpty(request.kpiCode))
        {
            AddComputedFields(resultData, sqlKpiDataList, request.kpiCode, kpiInfo);
        }

        return resultData;
    }

    private static (string, string, string, string) FormatDates(string startDate, string endDate)
    {
        var start = DateTime.Parse(startDate, CultureInfo.InvariantCulture);
        var end = DateTime.Parse(endDate, CultureInfo.InvariantCulture);

        return (
            start.ToString("yyyyMM"),
            end.ToString("yyyyMM"),
            start.ToString("yyyy-MM-dd"),
            end.ToString("yyyy-MM-dd")
        );
    }

    private static string FormatSapIds(string? sapIds)
    {
        if (string.IsNullOrWhiteSpace(sapIds)) return string.Empty;

        return string.Join(",", sapIds
            .Split(',')
            .Select(id => int.TryParse(id, out var num)
                ? (num.ToString().Length < 18 ? $"'000000000{num}'" : $"'{num}'")
                : $"'{id}'"));
    }

    private async Task<List<AssetBenchmarkGroupedData>> GetAssetBenchmarkDataAsync(
        GetAssetBenchmarkRequest request,
        string formattedStartDate,
        string formattedEndDate,
        string formattedStartDateYMD,
        string formattedEndDateYMD,
        string formattedSapIds,
        string quotedPmCode)
    {
        var queryList = new AssetBenchmarkQuery().plantDataList
            .Where(p => p.kpiCode == request.kpiCode)
            .ToList();

        var tasks = queryList.Select(item => FetchBenchmarkDataAsync(item, request,
            formattedStartDate, formattedEndDate,
            formattedStartDateYMD, formattedEndDateYMD,
            formattedSapIds, quotedPmCode));

        var resultSets = await Task.WhenAll(tasks);
        return resultSets.SelectMany(x => x).ToList();
    }

    private async Task<List<AssetBenchmarkGroupedData>> FetchBenchmarkDataAsync(
        AssetBenchmarkItem item,
        GetAssetBenchmarkRequest request,
        string startDate,
        string endDate,
        string startDateYMD,
        string endDateYMD,
        string sapIds,
        string quotedPmCode)
    {
        var parameters = new AssetQueryParameters
        {
            overalN = item.bestAchievedEver ?? string.Empty,
            formattedStartDate = startDate,
            formattedEndDate = endDate,
            formattedStartDateYyyyMmDd = startDateYMD,
            formattedEndDateYyyyMmDd = endDateYMD,
            affiliateRequest = string.Empty,
            request = request,
            quotedPmCode = quotedPmCode,
            sapIds = sapIds
        };

        var query = ReplaceAssetQuery(parameters);

        var results = await _benchMarkRepository.ExecuteBigDataQuery(query, MapReaderToAssetData);
        return results ?? new List<AssetBenchmarkGroupedData>();
    }

    private static AssetBenchmarkGroupedData MapReaderToAssetData(IDataReader reader)
    {
        return new AssetBenchmarkGroupedData
        {
            manufacturer = SafeGetString(reader, "manufacturer"),
            modelNumber = SafeGetString(reader, "model_number"),
            bestAchievedEver = SafeGetDecimal(reader, "best_achieved_ever"),
            absolute = SafeGetDecimal(reader, "absolute"),
            actual = SafeGetInt(reader, "actual"),
            sapId = SafeGetString(reader, "sap_id").TrimStart('0')
        };
    }

    private static string SafeGetString(IDataReader reader, string column)
        => reader[column] != DBNull.Value ? Convert.ToString(reader[column]) ?? "" : "";

    private static decimal SafeGetDecimal(IDataReader reader, string column)
        => reader[column] != DBNull.Value ? Convert.ToDecimal(reader[column]) : 0;

    private static int SafeGetInt(IDataReader reader, string column)
        => reader[column] != DBNull.Value ? Convert.ToInt32(reader[column]) : 0;

    private static void AddComputedFields(
        List<AssetBenchmarkGroupedData> groupData,
        List<KpiDataJsonResponse> kpiDataList,
        string kpiCode,
        KpiDataJsonResponse? kpiInfo)
    {
        if (groupData == null || groupData.Count == 0) return;

        var isAscending = kpiInfo?.direction == 1;

        var bestEverMin = isAscending
            ? groupData.Min(x => x.bestAchievedEver)
            : groupData.Max(x => x.bestAchievedEver);

        var bestSinglePeriod = isAscending
            ? groupData.Min(x => x.actual)
            : groupData.Max(x => x.actual);

        var direction = kpiDataList.FirstOrDefault(x => x.kpiCode == kpiCode)?.direction ?? 0;
        var targetMax = kpiDataList.FirstOrDefault(x => x.kpiCode == kpiCode)?.overallTargetMax ?? 0;

        foreach (var item in groupData)
        {
            item.direction = direction;
            item.target = (int)targetMax;
            item.bestAchievedEverMin = bestEverMin;
            item.bestAchievedForSinglePeriod = bestSinglePeriod;
        }
    }

    // Dummy method placeholders for missing references
    private static string ReplaceAssetQuery(AssetQueryParameters parameters) => string.Empty;
}
