public async Task<List<GroupedData>> GetAffDistFinalNumeratorDenominatorResult(List<KpiNumeratorDenominatorAffiliateDistribution> overAllNum, List<KpiNumeratorDenominatorAffiliateDistribution> overAllDen, List<KpiNumeratorDenominatorAffiliateDistribution> criticalNum, List<KpiNumeratorDenominatorAffiliateDistribution> criticalDen)
{
    var result = (from onum in overAllNum
                  join oden in overAllDen
                  on new { onum.kpiid, onum.affiliates }
                  equals new { oden.kpiid, oden.affiliates }
                  join cnum in criticalNum
                  on new { onum.kpiid, onum.affiliates }
                  equals new { cnum.kpiid, cnum.affiliates }
                  join cden in criticalDen
                  on new { onum.kpiid, onum.affiliates }
                  equals new { cden.kpiid, cden.affiliates }
                  select new KpiNumDenAffiliateDistributionResult
                  {
                      kpiid = onum.kpiid,
                      affiliates = onum.affiliates,
                      overallNum = onum.overallNumerator,
                      overallDen = oden.overallDenominator,
                      criticalNum = cnum.criticalNumerator,
                      criticalDen = cden.criticalDenominator,
                  }).ToList();


    var allaffiliates = await _currentRepository.GetAffiliateLists();

    var getaffiliatenamelist = (from r in result
                                join aff in allaffiliates
                                on r.affiliates equals aff.affiliateCode
                                select new
                                {
                                    r.kpiid,
                                    affiliatename = aff.affiliateName,
                                    r.overallNum,
                                    r.overallDen,
                                    r.criticalNum,
                                    r.criticalDen,
                                    r.affiliates
                                }
                      ).ToList();
    var finalResult = getaffiliatenamelist.GroupBy(x => new { x.kpiid, x.affiliatename }).//sum up the multiple affiliate numerator and denominator and get the actual value.
       Select(g => new KpiNumDenAffiliateDistributionResult

       {
           kpiid = g.Key.kpiid,
           affiliatename = g.Key.affiliatename,
           overallNum = g.Sum(x => x.overallNum),
           overallDen = g.Sum(x => x.overallDen),
           criticalNum = g.Sum(x => x.criticalNum),
           criticalDen = g.Sum(x => x.criticalDen),
           affiliates = g.Select(x => x.affiliates).FirstOrDefault()
       }).OrderBy(x => x.kpiid).ThenBy(x => x.affiliatename).ToList();

    List<GroupedData> lstResult = new List<GroupedData>();
    foreach (var item in finalResult)
    {
        GroupedData affDistResult = new GroupedData();
        affDistResult.affiliateId = Convert.ToString(item.affiliates);
        affDistResult.name = item.affiliatename;
        affDistResult.overall = item.overallDen != 0 ? ((item.overallNum / item.overallDen) * 100) : item.overallNum;
        affDistResult.critical = item.criticalDen != 0 ? ((item.criticalNum / item.criticalDen) * 100) : item.criticalNum;
        lstResult.Add(affDistResult);
    }
    return lstResult;
}
    #region GetAffDistFinalNumeratorDenominatorResult
    [Theory]
    [ClassData(typeof(GetAffDistFinalNumeratorDenominatorResultGenerator))]
    public async Task GetAffDistFinalNumeratorDenominatorResult_ShouldReturnData_WhenRepositoryReturnsData(List<KpiNumeratorDenominatorAffiliateDistribution> overAllNum,
        List<KpiNumeratorDenominatorAffiliateDistribution> overAllDen,
        List<KpiNumeratorDenominatorAffiliateDistribution> criticalNum,
        List<KpiNumeratorDenominatorAffiliateDistribution> criticalDen)
    {
        // Arrange
        var dbResponse = Enumerable.Range(0, 3).Select(index => Mock.Of<AffiliateList>())

.ToList();

        _currRepositoryMock.Setup(repo => repo.GetAffiliateLists()).ReturnsAsync(dbResponse);


        //Act
        var result = await _currServices.GetAffDistFinalNumeratorDenominatorResult(overAllNum, overAllDen, criticalNum, criticalDen);
        // Assert
        Assert.NotNull(result);
        Assert.IsType<List<GroupedData>>(result);
    }

    #endregion
