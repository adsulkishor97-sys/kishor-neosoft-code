public async Task<KpiPerformanceResponse> PerformanceSummaryAffiliateAsync(GetKpiPerformanceRequest request, string affiliateRequest)
{
    KpiPerformanceResponse kpiPerformanceResponse = new KpiPerformanceResponse();
    try { 
    var result = new KpiPerformanceResponse();
    result.kpis = new List<KpiDetail>();
    var allConvertedReports = new List<ConvertedKpiItemDetails>();
    var allConvertedplantReports = new List<ConvertedKpiItemPlantDetails>();


    KpiFormulaTargetRequest kpiFormulaTargetRequest = new KpiFormulaTargetRequest();
    GetCaseHierarchyRequest getCaseHierarchy = new GetCaseHierarchyRequest();
    getCaseHierarchy.affiliateIdList = request.affiliateId.ToString();

    List<GetCaseHierarchyResponse> plantDetails = await _configRepository.GetCaseHierarchyAsync(getCaseHierarchy);

    // get affiliate lists
    var affiliatesRes = _currentRepository.GetAffiliateLists().Result.ToList();
    kpiFormulaTargetRequest.performanceSummary = request.performanceSummary;

        var affiliateCodeList = affiliatesRes.Select(x => $"'{x.affiliateCode}'").ToList();
        var affiliateCodeString = string.Join(",", affiliateCodeList);
    // get KPIs details like formula based on performanceSummary 
    var kpisFormula = GetKpiFormulas(kpiFormulaTargetRequest);
    // get actual values from BigData Queries based on startDate,endDate and affiliate
    var actualData = GetPerformanceSummaryActualDataNew(request, affiliateCodeString);
    var actualPlantsData = GetPerformanceSummaryAffiliateActualData(request, affiliateRequest, plantDetails);

    // iterate loop to get affiliate list with actual and target values
    foreach (var kpi in kpisFormula)
    {
        
        var actualKpiname = kpi.name!.Replace(" ", "_");
        var report = await GenerateMaintenanceKpiReport(kpi.name!, affiliatesRes, actualData[actualKpiname!], kpi.target);
        var plantreport = await GenerateMaintenancePlantKpiReport(kpi.name!, plantDetails, actualPlantsData[actualKpiname!], kpi.target);
        var convertedReport = await GenerateConvertedMaintenanceReport(kpi!, report);
        var convertedPlantReport = await GenerateConvertedMaintenancePlantReport(kpi!, plantreport);


        result.kpis!.Add(report);
        result.kpis.Where(x => x.kpi == kpi.name).ToList().ForEach(x =>
        {
            x.plants ??= new List<Plant>();
            if (plantreport?.plants != null)
                x.plants.AddRange(plantreport.plants);
        }
        );

        allConvertedReports.AddRange(convertedReport);
        allConvertedplantReports.AddRange(convertedPlantReport);
        
    }

    // Calculate cost effectiveness
    decimal min = 0m;
    decimal max = 2m;
    var avgActualYs = allConvertedReports
        .GroupBy(c => c.affiliate!)
        .ToDictionary(g => g.Key, g => g.Sum(x => x.actualY));
    var avgPlantActualYs = allConvertedplantReports
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

    result.performanceSummary = kpiFormulaTargetRequest.performanceSummary;
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

    return result;
    }
    catch (Exception)
    {
        return kpiPerformanceResponse;
    }
}
public class GetKpiPerformanceRequest
{
    public string? startDate { get; set; }
    public string? endDate { get; set; }
    public string? performanceSummary { get; set; }
    public int? affiliateId { get; set; }
    public int? plantId { get; set; }
}
public class KpiPerformanceResponse
{
    public string? performanceSummary { get; set; } 
    public decimal average { get; set; }
    public decimal target { get; set; }
    public string? bestAffiliateName { get; set; }
    public decimal bestAffiliate { get; set; }
    public string? bestPlantName { get; set; }
    public decimal bestPlant { get; set; }
    public List<Affiliate>? affiliates { get; set; }
    public List<Plant>? plants { get; set; }
    public List<KpiDetail>? kpis { get; set; }
}
