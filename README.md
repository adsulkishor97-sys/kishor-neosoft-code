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

        KpiFormulaTargetRequest kpiFormulaTargetRequest = new KpiFormulaTargetRequest
        {
            performanceSummary = request.performanceSummary
        };

        GetCaseHierarchyRequest getCaseHierarchy = new GetCaseHierarchyRequest
        {
            affiliateIdList = request.affiliateId.ToString()
        };

        var plantDetails = await _configRepository.GetCaseHierarchyAsync(getCaseHierarchy) ?? new List<GetCaseHierarchyResponse>();
        var affiliatesRes = (await _currentRepository.GetAffiliateLists())?.ToList() ?? new List<AffiliateList>();

        var affiliateCodeList = affiliatesRes.Select(x => $"'{x.affiliateCode}'").ToList();
        var affiliateCodeString = string.Join(",", affiliateCodeList);

        var kpisFormula = GetKpiFormulas(kpiFormulaTargetRequest) ?? new List<KpiFormulaTarget>();
        var actualData = GetPerformanceSummaryActualDataNew(request, affiliateCodeString) ?? new Dictionary<string, Dictionary<int, decimal>>();
        var actualPlantsData = GetPerformanceSummaryAffiliateActualData(request, affiliateRequest, plantDetails) ?? new Dictionary<string, Dictionary<int, decimal>>();

        // ✅ Handle null or empty kpi formulas safely
        if (kpisFormula != null && kpisFormula.Any())
        {
            PopulateKpisAndCostEffectiveness(
                result,
                kpisFormula,
                affiliatesRes,
                plantDetails,
                actualData,
                actualPlantsData,
                kpiFormulaTargetRequest.performanceSummary ?? string.Empty);
        }

        return result;
    }
    catch (Exception)
    {
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
    if (result == null)
        return;

    // ✅ Defensive programming: if formulas are null or empty, stop early
    if (kpisFormula == null || !kpisFormula.Any())
    {
        result.kpis ??= new List<KpiDetail>();
        result.performanceSummary = performanceSummary;
        return;
    }

    var allConvertedReports = new List<ConvertedKpiItemDetails>();
    var allConvertedPlantReports = new List<ConvertedKpiItemPlantDetails>();

    foreach (var kpi in kpisFormula)
    {
        if (kpi?.kpi == null)
            continue;

        var actualKpiName = kpi.kpi.Replace(" ", "_");

        if (!actualData.ContainsKey(actualKpiName) || !actualPlantsData.ContainsKey(actualKpiName))
            continue;

        var report = GenerateMaintenanceKpiReport(kpi.kpi, affiliatesRes, actualData[actualKpiName], kpi.target).Result;
        var plantReport = GenerateMaintenancePlantKpiReport(kpi.kpi, plantDetails, actualPlantsData[actualKpiName], kpi.target).Result;
        var convertedReport = GenerateConvertedMaintenanceReport(kpi, report).Result;
        var convertedPlantReport = GenerateConvertedMaintenancePlantReport(kpi, plantReport).Result;

        result.kpis ??= new List<KpiDetail>();
        if (report != null)
            result.kpis.Add(report);

        var existing = result.kpis.Where(x => x.kpi == kpi.kpi).ToList();
        foreach (var x in existing)
        {
            x.plants ??= new List<Plant>();
            if (plantReport?.plants != null)
                x.plants.AddRange(plantReport.plants);
        }

        if (convertedReport != null)
            allConvertedReports.AddRange(convertedReport);
        if (convertedPlantReport != null)
            allConvertedPlantReports.AddRange(convertedPlantReport);
    }

    // ✅ Continue cost effectiveness logic unchanged
    decimal min = 0m;
    decimal max = 2m;

    var avgActualYs = allConvertedReports
        .GroupBy(c => c.affiliate!)
        .ToDictionary(g => g.Key, g => g.Sum(x => x.actualY));

    var avgPlantActualYs = allConvertedPlantReports
        .GroupBy(c => c.plant!)
        .ToDictionary(g => g.Key, g => g.Sum(x => x.actualY));

    var sumOfMaxAllAffiliates = allConvertedReports
        .GroupBy(c => c.affiliate!)
        .ToDictionary(g => g.Key, g => g.Sum(x => x.max));

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

    if (costEffectivenessPlantList.Count > 0)
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
