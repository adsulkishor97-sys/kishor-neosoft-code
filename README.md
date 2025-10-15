private static async Task<KpiDetail> GenerateMaintenanceKpiReport(string kpiName, List<AffiliateList> affiliates, Dictionary<int, decimal> actuals, decimal targetValue)
{
    KpiDetail datalst = new KpiDetail();
    try {
        var a = await Task.Run(() =>
        {
            var data = affiliates.Select(affiliate =>
            {
                actuals.TryGetValue(affiliate.affiliateCode, out var actualValue);
                var state = actualValue > targetValue ? 1 : 0;
                return new Affiliate
                {
                    affiliateName = affiliate.affiliateName,
                    actual = actualValue,
                    target = targetValue,
                    state = state
                };
            }).Where(a => a.actual > 0).ToList();


            // Best affiliate
            var best = data.OrderBy(r => r.actual).First();
            var bestAffiliate = new BestAffiliateDetails
            {
                affiliateName = best.affiliateName,
                actual = Math.Round(best.actual, 2),
                target = best.target,
                state = best.state
            };

            var average = (data.Sum(a => a.actual) / data.Count);

            return new KpiDetail
            {
                kpi = kpiName,
                bestAffiliate = Math.Round(bestAffiliate.actual, 2),
                bestAffiliateName = bestAffiliate.affiliateName,
                average = Math.Round(average, 2),
                target = bestAffiliate.target,
                affiliates = data.OrderBy(a => a.actual).ToList()
            };
        });
        return a;
    }
    catch (Exception ex) 
    {
        Console.WriteLine(ex.Message);
        return datalst;
    }
}
 public async Task<KpiPerformanceResponse> PerformanceSummaryAsync(GetKpiPerformanceRequest request, string affiliateRequest)
 {
     var result = new KpiPerformanceResponse();
     result.kpis = new List<KpiDetail>();
     var allConvertedReports = new List<ConvertedKpiItemDetails>();
     KpiFormulaTargetRequest kpiFormulaTargetRequest = new KpiFormulaTargetRequest();
     // get affiliate lists
     var affiliates = _currentRepository.GetAffiliateLists().Result.ToList();
     kpiFormulaTargetRequest.performanceSummary = request.performanceSummary;
     // get KPIs details like formula based on performanceSummary 
     var kpisFormula = GetKpiFormulas(kpiFormulaTargetRequest);
     // get actual values from BigData Queries based on startDate,endDate and affiliate
     var actualData = GetPerformanceSummaryActualDataNew(request, affiliateRequest);
     // iterate loop to get affiliate list with actual and target values
     foreach (var kpi in kpisFormula)
     {

         var actualKpiname = kpi.name!.Replace(" ", "_");
         try { 
         var report = await GenerateMaintenanceKpiReport(kpi.name!, affiliates, actualData[actualKpiname!], kpi.target);
         var convertedReport = await GenerateConvertedMaintenanceReport(kpi!, report);
        
         result.kpis!.Add(report);
         allConvertedReports.AddRange(convertedReport);
         }
         catch (Exception ex)
         {
             Console.WriteLine(ex);
         }

     }

     // Calculate cost effectiveness
     decimal min = 0m;
     decimal max = 2m;
     var avgActualYs = allConvertedReports
         .GroupBy(c => c.affiliate!)
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

     result.performanceSummary = kpiFormulaTargetRequest.performanceSummary;
     if(costEffectivenessList.Count>0)
     { 
     result.average = Math.Round(costEffectivenessList.Average(x => x.percentage), 2);
     result.bestAffiliate = Math.Round(costEffectivenessList.OrderByDescending(c => c.actual).FirstOrDefault()!.percentage, 2);
     result.bestAffiliateName = costEffectivenessList.OrderByDescending(c => c.actual).FirstOrDefault()!.affiliate;
     result.target = costEffectivenessList.OrderByDescending(c => c.target).FirstOrDefault()!.target;
     result.affiliates = costEffectivenessList.Select(kvp => new Affiliate
     {
         affiliateName = kvp.affiliate,
         target = kvp.target,
         actual = Math.Round(kvp.percentage, 2)

     }).ToList();
     }

     return result;
 }
 #region PerformanceSummaryAsync
[Theory]
[ClassData(typeof(PerformanceSummaryRequestGenerator))]
public async Task PerformanceSummaryAsync_ShouldReturnData_WhenRepositoryReturnsData(GetKpiPerformanceRequest request, GetCaseHierarchyRequest affiliateIdrequest, List<AffiliateList> affiliateList
    , List<KpiFormulaTarget> dbFormulas)
{
    // Arrange    

    _currRepositoryMock.Setup(repo => repo.GetAffiliateLists()).ReturnsAsync(affiliateList);

    _performanceSumRepositoryMock.Setup(repo => repo.GetKpiFormulaTarget(It.IsAny<KpiFormulaTargetRequest>())).Returns(Task.FromResult(dbFormulas));
    //Act
    var result = await _performanceSummServices.PerformanceSummaryAsync(request, affiliateIdrequest.affiliateIdList!);
    // Assert
    Assert.NotNull(result);
    Assert.IsType<KpiPerformanceResponse>(result);

}

#endregion
