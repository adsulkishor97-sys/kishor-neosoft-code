public async Task<List<GroupedData>> GetAffDistFinalOverallNumDenPlantResult(List<KpiNumeratorDenominatorAffiliateDistribution> overAllNum, List<KpiNumeratorDenominatorAffiliateDistribution> overAllDen)
{
    var result = (from onum in overAllNum
                  join oden in overAllDen
                  on new { onum.kpiid, onum.plantid }
                  equals new { oden.kpiid, oden.plantid }
                  select new KpiNumDenAffiliateDistributionResult
                  {
                      kpiid = onum.kpiid,
                      plant = onum.plantid,
                      overallNum = onum.overallNumerator,
                      overallDen = oden.overallDenominator,
                  }).ToList();
    var allplants = await _currentRepository.GetPlantLists();

    var getaffiliatenamelist = (from r in result
                                join plnt in allplants
                                on r.plant equals plnt.plantId
                                select new
                                {
                                    r.kpiid,
                                    plantname = plnt.plantName,
                                    r.overallNum,
                                    r.overallDen,
                                    r.plant
                                }
                      ).ToList();
    var finalResult = getaffiliatenamelist.GroupBy(x => new { x.kpiid, x.plantname }).//sum up the multiple affiliate numerator and denominator and get the actual value.
       Select(g => new KpiNumDenAffiliateDistributionResult

       {
           kpiid = g.Key.kpiid,
           plantname = g.Key.plantname.ToString(),
           overallNum = g.Sum(x => x.overallNum),
           criticalNum = g.Sum(x => x.overallDen),
           plant = g.Select(x => x.plant).FirstOrDefault()
       }).OrderBy(x => x.kpiid).ThenBy(x => x.plantname).ToList();

    List<GroupedData> lstResult = new List<GroupedData>();
    foreach (var item in finalResult)
    {
        GroupedData affDistResult = new GroupedData();
        affDistResult.plantId = Convert.ToString(item.plant);
        affDistResult.plantName = item.plantname;
        affDistResult.overall = item.overallDen != 0 ? ((item.overallNum / item.overallDen) * 100) : item.overallNum;
        affDistResult.critical = item.criticalDen != 0 ? ((item.criticalNum / item.criticalDen) * 100) : item.criticalNum;
        lstResult.Add(affDistResult);
    }
    return lstResult;
}
