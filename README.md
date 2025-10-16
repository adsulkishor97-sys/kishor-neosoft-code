public async Task<KpiPerformanceResponse> PerformanceSummaryAffiliateAsync(GetKpiPerformanceRequest request, string affiliateRequest)
{
    KpiPerformanceResponse kpiPerformanceResponse = new KpiPerformanceResponse();

    try
    {
        var result = new KpiPerformanceResponse
        {
            kpis = new List<KpiDetail>()
        };

        if (request == null)
            return kpiPerformanceResponse;

        var kpiFormulaTargetRequest = new KpiFormulaTargetRequest
        {
            performanceSummary = request.performanceSummary
        };

        var getCaseHierarchy = new GetCaseHierarchyRequest
        {
            affiliateIdList = request.affiliateId.ToString()
        };

        // Fetch data safely
        var plantDetails = await _configRepository.GetCaseHierarchyAsync(getCaseHierarchy) ?? new List<GetCaseHierarchyResponse>();
        var affiliatesRes = (await _currentRepository.GetAffiliateLists())?.ToList() ?? new List<AffiliateList>();

        var affiliateCodeList = affiliatesRes?.Select(x => $"'{x.affiliateCode}'").ToList() ?? new List<string>();
        var affiliateCodeString = string.Join(",", affiliateCodeList);

        // Protected/private data fetch methods
        var kpisFormula = GetKpiFormulas(kpiFormulaTargetRequest) ?? new List<KpiFormulaTarget>();
        var actualData = GetPerformanceSummaryActualDataNew(request, affiliateCodeString) ?? new Dictionary<string, Dictionary<int, decimal>>();
        var actualPlantsData = GetPerformanceSummaryAffiliateActualData(request, affiliateRequest, plantDetails) ?? new Dictionary<string, Dictionary<int, decimal>>();

        // Safely process all
        PopulateKpisAndCostEffectiveness(result, kpisFormula, affiliatesRes, plantDetails, actualData, actualPlantsData, kpiFormulaTargetRequest.performanceSummary ?? string.Empty);

        return result;
    }
    catch (Exception)
    {
        // In case of any unexpected error, return default response safely
        return kpiPerformanceResponse;
    }
}

private void PopulateKpisAndCostEffectiveness(
    KpiPerformanceResponse result,
    List<KpiFormulaTarget> kpisFormula,
    List<AffiliateList> affiliatesRes,
    List<GetCaseHierarchyResponse> plantDetails,
    Dictionary<string, Dictionary<int, decimal>> actualData,
    Dictionary<string, Dictionary<int, decimal>> actualPlantsData,
    string performanceSummary)
{
    var allConvertedReports = new List<ConvertedKpiItemDetails>();
    var allConvertedPlantReports = new List<ConvertedKpiItemPlantDetails>();

    if (kpisFormula == null || !kpisFormula.Any())
        return;

    foreach (var kpi in kpisFormula)
    {
        if (string.IsNullOrWhiteSpace(kpi?.name))
            continue;

        var actualKpiName = kpi.name.Replace(" ", "_");

        if (!actualData.ContainsKey(actualKpiName) || !actualPlantsData.ContainsKey(actualKpiName))
            continue;

        // Protected/async method calls
        var report = GenerateMaintenanceKpiReport(kpi.name, affiliatesRes, actualData[actualKpiName], kpi.target).Result;
        var plantReport = GenerateMaintenancePlantKpiReport(kpi.name, plantDetails, actualPlantsData[actualKpiName], kpi.target).Result;
        var convertedReport = GenerateConvertedMaintenanceReport(kpi, report).Result;
        var convertedPlantReport = GenerateConvertedMaintenancePlantReport(kpi, plantReport).Result;

        if (report != null)
            result.kpis?.Add(report);

        var existing = result.kpis?.Where(x => x.kpi == kpi.name).ToList();
        if (existing != null && existing.Any())
        {
            foreach (var x in existing)
            {
                x.plants ??= new List<Plant>();
                if (plantReport?.plants != null)
                    x.plants.AddRange(plantReport.plants);
            }
        }

        if (convertedReport != null)
            allConvertedReports.AddRange(convertedReport);
        if (convertedPlantReport != null)
            allConvertedPlantReports.AddRange(convertedPlantReport);
    }

    // Cost effectiveness
    decimal min = 0m;
    decimal max = 2m;

    var avgActualYs = allConvertedReports?
        .Where(c => c?.affiliate != null)
        .GroupBy(c => c.affiliate!)
        .ToDictionary(g => g.Key, g => g.Sum(x => x.actualY)) ?? new Dictionary<string, decimal>();

    var avgPlantActualYs = allConvertedPlantReports?
        .Where(c => c?.plant != null)
        .GroupBy(c => c.plant!)
        .ToDictionary(g => g.Key, g => g.Sum(x => x.actualY)) ?? new Dictionary<string, decimal>();

    var sumOfMaxAllAffiliates = allConvertedReports?
        .Where(c => c?.affiliate != null)
        .GroupBy(c => c.affiliate!)
        .ToDictionary(g => g.Key, g => g.Sum(x => x.max)) ?? new Dictionary<string, decimal>();

    if (sumOfMaxAllAffiliates.Any())
        max = sumOfMaxAllAffiliates.Select(a => a.Value).FirstOrDefault();

    var costEffectivenessList = avgActualYs
        .Select(kvp => new PerformanceSummaryGroupItem(
            kvp.Key,
            kvp.Value,
            min,
            max,
            max > 0 ? (kvp.Value / max) * 100m : 0m,
            100m))
        .ToList();

    var costEffectivenessPlantList = avgPlantActualYs
        .Select(kvp => new PerformanceSummaryPlantGroupItem(
            kvp.Key,
            kvp.Value,
            min,
            max,
            max > 0 ? (kvp.Value / max) * 100m : 0m,
            100m))
        .ToList();

    result.performanceSummary = performanceSummary;

    if (costEffectivenessPlantList.Count > 0 && costEffectivenessList.Count > 0)
    {
        result.average = Math.Round(costEffectivenessList.Average(x => x.percentage), 2);
        result.bestAffiliate = Math.Round(costEffectivenessList.OrderByDescending(c => c.actual).FirstOrDefault()?.percentage ?? 0, 2);
        result.bestAffiliateName = costEffectivenessList.OrderByDescending(c => c.actual).FirstOrDefault()?.affiliate ?? string.Empty;
        result.target = costEffectivenessList.OrderByDescending(c => c.target).FirstOrDefault()?.target ?? 0;
        result.plants = costEffectivenessPlantList.Select(kvp => new Plant
        {
            plantName = kvp.Plant,
            target = kvp.Target,
            actual = Math.Round(kvp.Percentage, 2)
        }).ToList();
    }

    result.kpis?.ForEach(x => x.affiliates = null);
    result.affiliates = null;
}
