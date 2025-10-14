 public async Task<KpiPerformanceResponse> PerformanceSummaryPlantAsync(GetKpiPerformanceRequest request, string affiliateRequest)
 {
     KpiPerformanceResponse kpiPerformanceResponse = new KpiPerformanceResponse();

     try
     {
         var result = new KpiPerformanceResponse();
         result.kpis = new List<KpiDetail>();
         var allConvertedReport = new List<ConvertedKpiItemDetails>();
         var allConvertedplantReports = new List<ConvertedKpiItemPlantDetails>();

         KpiFormulaTargetRequest kpiFormulaTargetRequest = new KpiFormulaTargetRequest();
         GetCaseHierarchyRequest getCaseHierarchy = new GetCaseHierarchyRequest();
         getCaseHierarchy.affiliateIdList = request.affiliateId.ToString();

         List<GetCaseHierarchyResponse> plantDetails = await _configRepository.GetCaseHierarchyAsync(getCaseHierarchy);

         // get affiliate lists
         var affiliatesRes = _currentRepository.GetAffiliateLists().Result.ToList();
         var affiliateDataResult = _currentRepository.GetAffiliateLists().Result.Where(x => x.affiliateId == request.affiliateId).ToList();
         string formatedAffiliateData = string.Join(",", affiliateDataResult.Select(x => $"'{x.affiliateCode}'").ToList());
         kpiFormulaTargetRequest.performanceSummary = request.performanceSummary;

         // get KPIs details like formula based on performanceSummary 
         var kpisFormula = GetKpiFormulas(kpiFormulaTargetRequest);
         // get actual values from BigData Queries based on startDate,endDate and affiliate
         var actualData = GetPerformanceSummaryActualDataNew(request, affiliateRequest);
         var actualPlantsData = GetPerformanceSummaryAffiliateActualData(request, formatedAffiliateData, plantDetails);

         // iterate loop to get affiliate list with actual and target values
         foreach (var kpi in kpisFormula)
         {

             var actualKpiname = kpi.name!.Replace(" ", "_");
             var report = await GenerateMaintenanceKpiReport(kpi.name!, affiliatesRes, actualData[actualKpiname!], kpi.target);
             var plantreport = await GenerateMaintenancePlantKpiReport(kpi.name!, plantDetails, actualPlantsData[actualKpiname!], kpi.target);
             var convertedReport = await GenerateConvertedMaintenanceReport(kpi!, report);
             var convertedPlantReport = await GenerateConvertedMaintenancePlantReport(kpi!, plantreport);


             result.kpis!.Add(report);
             result.kpis.ForEach(x =>
             {
                 x.plants ??= new List<Plant>();
                 if (plantreport?.plants != null)
                     x.plants.AddRange(plantreport.plants);
             }
             );

             allConvertedReport.AddRange(convertedReport);
             allConvertedplantReports.AddRange(convertedPlantReport);

         }

         // Calculate cost effectiveness
         decimal min = 0m;
         decimal max = 2m;
         var avgActualYs = allConvertedReport
             .GroupBy(c => c.affiliate!)
             .ToDictionary(g => g.Key, g => g.Sum(x => x.actualY));
         var avgPlantActualYs = allConvertedplantReports
            .GroupBy(c => c.plant!)
            .ToDictionary(g => g.Key, g => g.Sum(x => x.actualY));

         var sumOfMaxAllAffiliates = allConvertedReport
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
         Console.WriteLine(costEffectivenessList);

         var costEffectivenessPlantList = avgPlantActualYs
             .Select(kvp => new PerformanceSummaryPlantGroupItem(
                 kvp.Key,
                 kvp.Value,
                 min,
                 max,
                 (kvp.Value / max) * 100m,
                 100m))
             .ToList();
         string? PlantName = plantDetails!.Where(x => x.plantId == request.plantId).Select(x => x.plantName).FirstOrDefault();
         result.performanceSummary = kpiFormulaTargetRequest.performanceSummary;
         if (costEffectivenessPlantList.Count > 0)
         {
             result.average = Math.Round(costEffectivenessPlantList.Where(x => x.Plant == PlantName).Average(x => x.Percentage), 2);
             result.bestPlant = Math.Round(costEffectivenessPlantList.OrderByDescending(c => c.Actual).FirstOrDefault()!.Percentage, 2);
             result.bestPlantName = costEffectivenessPlantList.OrderByDescending(c => c.Actual).FirstOrDefault()!.Plant;
             result.target = costEffectivenessPlantList.OrderByDescending(c => c.Target).FirstOrDefault()!.Target;
         }
         result.plants = null;
         result.kpis.ForEach(x => x.affiliates = null);
         result.affiliates = null;
         result.kpis = null;

         return result;
     }
     catch (Exception)
     {
         return kpiPerformanceResponse;
     }
 }
