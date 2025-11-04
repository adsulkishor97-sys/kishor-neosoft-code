public async Task<List<GroupedData>> GetAffDistFinalOverallCriticalResult(List<KpiNumeratorDenominatorAffiliateDistribution> overAllCriticalResult)
{
    var allaffiliates = await _currentRepository.GetAffiliateLists();

    var getaffiliatenamelist = (from r in overAllCriticalResult
                                join aff in allaffiliates
                                on r.affiliates equals aff.affiliateCode
                                select new
                                {
                                    r.kpiid,
                                    affiliatename = aff.affiliateName,
                                    r.overallNumerator,
                                    r.criticalNumerator,
                                    r.affiliates
                                }
                      ).ToList();
    var finalResult = getaffiliatenamelist.GroupBy(x => new { x.kpiid, x.affiliatename }).//sum up the multiple affiliate numerator and denominator and get the actual value.
       Select(g => new KpiNumDenAffiliateDistributionResult

       {
           kpiid = g.Key.kpiid,
           affiliatename = g.Key.affiliatename,
           overallNum = g.Average(x => x.overallNumerator),
           criticalNum = g.Average(x => x.criticalNumerator),
           affiliates = g.Select(x => x.affiliates).FirstOrDefault()
       }).OrderBy(x => x.kpiid).ThenBy(x => x.affiliatename).ToList();

    List<GroupedData> lstResult = new List<GroupedData>();
    foreach (var item in finalResult)
    {
        GroupedData affDistResult = new GroupedData();
        affDistResult.affiliateId = Convert.ToString(item.affiliates);
        affDistResult.name = item.affiliatename;
        affDistResult.overall = item.overallNum;
        affDistResult.critical = item.criticalNum;
        lstResult.Add(affDistResult);
    }
    return lstResult;
}
