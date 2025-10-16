public async Task<KpiPerformanceResponse> PerformanceSummaryAffiliateAsync(GetKpiPerformanceRequest request, string affiliateRequest)
{
    KpiPerformanceResponse kpiPerformanceResponse = new KpiPerformanceResponse();
    try
    {
        var result = new KpiPerformanceResponse
        {
            kpis = new List<KpiDetail>()
        };

        KpiFormulaTargetRequest kpiFormulaTargetRequest = new KpiFormulaTargetRequest();
        GetCaseHierarchyRequest getCaseHierarchy = new GetCaseHierarchyRequest
        {
            affiliateIdList = request.affiliateId.ToString()
        };

        var plantDetails = await _configRepository.GetCaseHierarchyAsync(getCaseHierarchy);
        var affiliatesRes = _currentRepository.GetAffiliateLists().Result.ToList();

        kpiFormulaTargetRequest.performanceSummary = request.performanceSummary;

        var affiliateCodeList = affiliatesRes.Select(x => $"'{x.affiliateCode}'").ToList();
        var affiliateCodeString = string.Join(",", affiliateCodeList);

        var kpisFormula = GetKpiFormulas(kpiFormulaTargetRequest);
        var actualData = GetPerformanceSummaryActualDataNew(request, affiliateCodeString);
        var actualPlantsData = GetPerformanceSummaryAffiliateActualData(request, affiliateRequest, plantDetails);

        // Call the extracted private method
        PopulateKpisAndCostEffectiveness(result, kpisFormula, affiliatesRes, plantDetails, actualData, actualPlantsData, kpiFormulaTargetRequest.performanceSummary);

        return result;
    }
    catch (Exception)
    {
        return kpiPerformanceResponse;
    }
}

// Extracted private helper
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

    foreach (var kpi in kpisFormula)
    {
        var actualKpiname = kpi.name!.Replace(" ", "_");
        var report = GenerateMaintenanceKpiReport(kpi.name!, affiliatesRes, actualData[actualKpiname!], kpi.target).Result;
        var plantreport = GenerateMaintenancePlantKpiReport(kpi.name!, plantDetails, actualPlantsData[actualKpiname!], kpi.target).Result;
        var convertedReport = GenerateConvertedMaintenanceReport(kpi, report).Result;
        var convertedPlantReport = GenerateConvertedMaintenancePlantReport(kpi, plantreport).Result;

        result.kpis!.Add(report);
        result.kpis.Where(x => x.kpi == kpi.name).ToList().ForEach(x =>
        {
            x.plants ??= new List<Plant>();
            if (plantreport?.plants != null)
                x.plants.AddRange(plantreport.plants);
        });

        allConvertedReports.AddRange(convertedReport);
        allConvertedPlantReports.AddRange(convertedPlantReport);
    }

    // Calculate cost effectiveness
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
            (kvp.Value / max) * 100m,
            100m))
        .ToList();

    var costEffectivenessPlantList = avgPlantActualYs
        .Select(kvp => new PerformanceSummaryPlantGroupItem(
            kvp.Key,
            kvp.Value,
            min,
            max,
            (kvp.Value / max) * 100m,
            100m))
        .ToList();

    result.performanceSummary = performanceSummary;

    if (costEffectivenessPlantList.Count > 0)
    {
        result.average = Math.Round(costEffectivenessList.Average(x => x.percentage), 2);
        result.bestAffiliate = Math.Round(costEffectivenessList.OrderByDescending(c => c.actual).FirstOrDefault()!.percentage, 2);
        result.bestAffiliateName = costEffectivenessList.OrderByDescending(c => c.actual).FirstOrDefault()!.affiliate;
        result.target = costEffectivenessList.OrderByDescending(c => c.target).FirstOrDefault()!.target;
        result.plants = costEffectivenessPlantList.Select(kvp => new Plant
        {
            plantName = kvp.Plant,
            target = kvp.Target,
            actual = Math.Round(kvp.Percentage, 2)
        }).ToList();
    }

    result.kpis.ForEach(x => x.affiliates = null);
    result.affiliates = null;
}
