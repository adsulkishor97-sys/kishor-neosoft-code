using System;
using System.Collections.Generic;
using System.Data;
using System.Globalization;
using System.Linq;
using System.Threading.Tasks;

public class AssetService
{
    private readonly IBenchMarkRepository _benchMarkRepository;

    public AssetService(IBenchMarkRepository benchMarkRepository)
    {
        _benchMarkRepository = benchMarkRepository;
    }

    public async Task<List<AssetBenchmarkGroupedData>> GetAssetBenchmark(GetAssetBenchmarkRequest request)
    {
        var (formatedStartdate, formatedEnddate, formatedStartdateyymmdd, formatedEnddateyymmdd) =
            FormatDatesAsset(request.startDate!, request.endDate!);

        var formatSapIds = FormatSapIds(request.sapId);

        var sqlKpiDataList = await _benchMarkRepository.GetBusinessKPIDetails();
        var kpiinfo = sqlKpiDataList.Find(x => x.kpiCode == request.kpiCode);

        string quatedpmcode = "";

        var groupresultdata = await GetAssetBenchmarkDataAsync(
            request,
            formatedStartdate,
            formatedEnddate,
            formatedStartdateyymmdd,
            formatedEnddateyymmdd,
            formatSapIds,
            quatedpmcode);

        if (groupresultdata == null || groupresultdata.Count == 0)
            return new List<AssetBenchmarkGroupedData>();

        if (!string.IsNullOrEmpty(request.kpiCode))
        {
            AddAssetComputedFields(groupresultdata, sqlKpiDataList, request.kpiCode, kpiinfo);
        }

        return groupresultdata;
    }

    private static (string, string, string, string) FormatDatesAsset(string startDate, string endDate)
    {
        var startDateTime = DateTime.Parse(startDate, CultureInfo.InvariantCulture);
        var endDateTime = DateTime.Parse(endDate, CultureInfo.InvariantCulture);

        return (
            startDateTime.ToString("yyyyMM"),
            endDateTime.ToString("yyyyMM"),
            startDateTime.ToString("yyyy-MM-dd"),
            endDateTime.ToString("yyyy-MM-dd")
        );
    }

    private static string FormatSapIds(string? sapIds)
    {
        var sapList = sapIds?.Split(',').Select(int.Parse).ToList() ?? new List<int>();
        return string.Join(",", sapList.Select(x => x.ToString().Length < 18 ? $"'000000000{x}'" : $"'{x}'"));
    }

    private async Task<List<AssetBenchmarkGroupedData>> GetAssetBenchmarkDataAsync(
        GetAssetBenchmarkRequest request,
        string formatedStartdate,
        string formatedEnddate,
        string formatedStartdateyymmdd,
        string formatedEnddateyymmdd,
        string formatSapIds,
        string quatedpmcode)
    {
        var assetBenchmarkQuery = new AssetBenchmarkQuery();
        var plantDataList = assetBenchmarkQuery.plantDataList
            .Where(a => a.kpiCode == request.kpiCode)
            .ToList();

        var tasks = plantDataList
            .Select(item => GetAssetBenchmarkForItemAsync(item, request, formatedStartdate, formatedEnddate,
                formatedStartdateyymmdd, formatedEnddateyymmdd, formatSapIds, quatedpmcode));

        var results = await Task.WhenAll(tasks);
        return results.SelectMany(x => x).ToList();
    }

    private async Task<List<AssetBenchmarkGroupedData>> GetAssetBenchmarkForItemAsync(
        AffiliateBenchmark item,
        GetAssetBenchmarkRequest request,
        string formatedStartdate,
        string formatedEnddate,
        string formatedStartdateyymmdd,
        string formatedEnddateyymmdd,
        string formatSapIds,
        string quatedpmcode)
    {
        var parameters = new AssetQueryParameters
        {
            overalN = item.bestAchievedEver ?? string.Empty,
            formattedStartDate = formatedStartdate,
            formattedEndDate = formatedEnddate,
            formattedStartDateYyyyMmDd = formatedStartdateyymmdd,
            formattedEndDateYyyyMmDd = formatedEnddateyymmdd,
            affiliateRequest = string.Empty,
            request = request,
            quotedPmCode = quatedpmcode,
            sapIds = formatSapIds
        };

        var query = ReplaceAssetQuery(parameters);

        return await _benchMarkRepository.ExecuteBigDataQuery<AssetBenchmarkGroupedData>(query, MapAssetBenchmarkGroupedData)
               ?? new List<AssetBenchmarkGroupedData>();
    }

    private static AssetBenchmarkGroupedData MapAssetBenchmarkGroupedData(IDataRecord reader)
    {
        return new AssetBenchmarkGroupedData
        {
            manufacturer = reader["manufacturer"] != DBNull.Value ? Convert.ToString(reader["manufacturer"]) : "",
            modelNumber = reader["model_number"] != DBNull.Value ? Convert.ToString(reader["model_number"]) : "",
            bestAchievedEver = reader["best_achieved_ever"] != DBNull.Value ? Convert.ToDecimal(reader["best_achieved_ever"]) : 0,
            absolute = reader["absolute"] != DBNull.Value ? Convert.ToDecimal(reader["absolute"]) : 0,
            actual = reader["actual"] != DBNull.Value ? Convert.ToInt32(reader["actual"]) : 0,
            sapId = reader["sap_id"] != DBNull.Value ? Convert.ToString(reader["sap_id"])!.TrimStart('0') : "",
        };
    }

    private static string ReplaceAssetQuery(AssetQueryParameters parameters)
    {
        return parameters.overalN!
            .Replace("StartDateyyyymmdd", parameters.formattedStartDateYyyyMmDd)
            .Replace("EndDateyyyymmdd", parameters.formattedEndDateYyyyMmDd)
            .Replace("affiliateRequest", parameters.affiliateRequest)
            .Replace("${startDateyyyymm}", $"'{parameters.formattedStartDate}'")
            .Replace("${endDateyyyymm}", $"'{parameters.formattedEndDate}'")
            .Replace("${startDate}", $"'{parameters.formattedStartDate}'")
            .Replace("${endDate}", $"'{parameters.formattedEndDate}'")
            .Replace("${sapId}", parameters.sapIds);
    }

    private static void AddAssetComputedFields(
        List<AssetBenchmarkGroupedData> groupData,
        List<KpiDataJsonResponse> sqlKpiDataList,
        string kpiCode,
        KpiDataJsonResponse? kpiinfo)
    {
        decimal bestAchivedEverMin = kpiinfo?.direction == 1
            ? groupData.Min(x => x.bestAchievedEver)
            : groupData.Max(x => x.bestAchievedEver);

        decimal bestAchivedForSinglePeriod = kpiinfo?.direction == 1
            ? groupData.Min(x => x.actual)
            : groupData.Max(x => x.actual);

        int? sqldirection = sqlKpiDataList.Where(x => x.kpiCode == kpiCode).Select(x => x.direction).FirstOrDefault();
        long? targetMax = sqlKpiDataList.Where(x => x.kpiCode == kpiCode).Select(x => x.overallTargetMax).FirstOrDefault();

        foreach (var itemData in groupData)
        {
            itemData.direction = sqldirection ?? 0;
            itemData.target = (int)(targetMax ?? 0);
            itemData.bestAchievedEverMin = bestAchivedEverMin;
            itemData.bestAchievedForSinglePeriod = bestAchivedForSinglePeriod;
        }
    }
}
