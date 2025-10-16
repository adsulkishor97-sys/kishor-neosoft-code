 private async Task<List<KpiNumeratorDenominatorPerformanceSummary>> GetPerformanceSummaryActualDataNew(GetKpiPerformanceRequest request, DateTimeFormatRequest dateTimeFormatRequest)
 {
     List<KpiNumeratorDenominatorPerformanceSummary> bigDataList = new List<KpiNumeratorDenominatorPerformanceSummary>();


     //here get  json file data           

     List<MaintananceExcellence> responseList = new();
     GlobalPerformanceSummary globalPerformanceSummary = new GlobalPerformanceSummary();
     if (request.performanceSummary == KpiConstant.MaintananceExcellence)
     {

         responseList = globalPerformanceSummary.maintananceExcellence.ToList();

     }
     else if (request.performanceSummary == KpiConstant.CostEffectivness)
     {

         responseList = globalPerformanceSummary.costEffectiveness.ToList();


     }
     else if (request.performanceSummary == KpiConstant.AssetPerformance)
     {

         responseList = globalPerformanceSummary.assetPerformance.ToList();



     }
     List<Task> tasks = new List<Task>();


     foreach (var item in responseList.ToList())
     {
         tasks.Add(Task.Run(async () =>
         {

             var queryNumerator = item.Numerator!
.Replace("StartDateyyyymmdd", dateTimeFormatRequest.startDateyyyymmdd)
.Replace("EndDateyyyymmdd", dateTimeFormatRequest.endDateyyyymmdd)
.Replace("affiliateRequest", dateTimeFormatRequest.affiliateRequest)
.Replace("StartDateyyyymm", dateTimeFormatRequest.startDateyyyymm)
.Replace("EndDateyyyymm", dateTimeFormatRequest.endDateyyyymm);

             var queryDenominator = item.Denominator!
                        .Replace("StartDateyyyymmdd", dateTimeFormatRequest.startDateyyyymmdd)
                        .Replace("EndDateyyyymmdd", dateTimeFormatRequest.endDateyyyymmdd)
                        .Replace("affiliateRequest", dateTimeFormatRequest.affiliateRequest)
                        .Replace("StartDateyyyymm", dateTimeFormatRequest.startDateyyyymm)
                        .Replace("EndDateyyyymm", dateTimeFormatRequest.endDateyyyymm);


             var result = await GetPerformanceSummaryNumDenCalc(queryNumerator!, queryDenominator!);

             bigDataList.AddRange(result);


         }));
     }

     await Task.WhenAll(tasks);

     return bigDataList;

 }
