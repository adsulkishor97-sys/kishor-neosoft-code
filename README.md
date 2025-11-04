public async Task<List<AssetGroupedData>> GetAffDistFinalOverallNumAssetResult(List<KpiNumeratorDenominatorAffiliateDistribution> overAllNum, string category)
{
    dynamic finalResult;
    var allplants = await _currentRepository.GetPlantLists();

    var getaffiliatenamelist = (from r in overAllNum
                                join plnt in allplants
                                on r.plantid equals plnt.plantId
                                select new
                                {
                                    r.kpiid,
                                    plantname = plnt.plantName,
                                    r.overallNumerator,
                                    r.sap_id,
                                    r.template,
                                    r.tidnr,
                                    r.pltxt,
                                    r.plantid
                                }
                      ).ToList();
    if (category == "U")
    {
        finalResult = getaffiliatenamelist.GroupBy(x => new { x.kpiid, x.pltxt }).//sum up the multiple affiliate numerator and denominator and get the actual value.
           Select(g => new KpiNumDenAffiliateDistributionResult

           {
               kpiid = g.Key.kpiid,
               overallNum = g.Sum(x => x.overallNumerator),
               plant = g.Select(x => x.plantid).FirstOrDefault(),
               sapId = g.Select(x => x.sap_id).FirstOrDefault(),
               template = g.Select(x => x.template).FirstOrDefault(),
               tidnr = g.Select(x => x.tidnr).FirstOrDefault(),
               pltxt = g.Select(x => x.pltxt).FirstOrDefault(),
           }).OrderBy(x => x.kpiid).ThenBy(x => x.plantname).ToList();
    }
    else if (category == "AC")
    {
        finalResult = getaffiliatenamelist.GroupBy(x => new { x.kpiid, x.template }).//sum up the multiple affiliate numerator and denominator and get the actual value.
           Select(g => new KpiNumDenAffiliateDistributionResult

           {
               kpiid = g.Key.kpiid,
               overallNum = g.Sum(x => x.overallNumerator),
               plant = g.Select(x => x.plantid).FirstOrDefault(),
               sapId = g.Select(x => x.sap_id).FirstOrDefault(),
               template = g.Select(x => x.template).FirstOrDefault(),
               tidnr = g.Select(x => x.tidnr).FirstOrDefault(),
               pltxt = g.Select(x => x.pltxt).FirstOrDefault(),
           }).OrderBy(x => x.kpiid).ThenBy(x => x.plantname).ToList();
    }
    else
    {
        finalResult = getaffiliatenamelist.GroupBy(x => new { x.kpiid, x.tidnr }).//sum up the multiple affiliate numerator and denominator and get the actual value.
           Select(g => new KpiNumDenAffiliateDistributionResult

           {
               kpiid = g.Key.kpiid,
               overallNum = g.Sum(x => x.overallNumerator),
               plant = g.Select(x => x.plantid).FirstOrDefault(),
               sapId = g.Select(x => x.sap_id).FirstOrDefault(),
               template = g.Select(x => x.template).FirstOrDefault(),
               tidnr = g.Select(x => x.tidnr).FirstOrDefault(),
               pltxt = g.Select(x => x.pltxt).FirstOrDefault(),
           }).OrderBy(x => x.kpiid).ThenBy(x => x.plantname).ToList();
    }

    List<AssetGroupedData> lstResult = new List<AssetGroupedData>();
    foreach (var item in finalResult)
    {
        AssetGroupedData assetDistResult = new AssetGroupedData();
        assetDistResult.plantId = Convert.ToString(item.plant);
        assetDistResult.overall = item.overallNum;
        assetDistResult.sapId = item.sapId;
        assetDistResult.template = item.template;
        assetDistResult.tidnr = item.tidnr;
        assetDistResult.pltxt = item.pltxt;
        lstResult.Add(assetDistResult);
    }
    return lstResult;
}
    #region GetAffDistFinalOverallNumAssetResult
    [Theory]
    [ClassData(typeof(GetAffDistFinalOverallNumAssetResultGenerator))]
    public async Task GetAffDistFinalOverallNumAssetResult_ShouldReturnData_WhenRepositoryReturnsData(List<KpiNumeratorDenominatorAffiliateDistribution> overAllNum, string category)
    {
        // Arrange
        var dbResponse = Enumerable.Range(0, 3).Select(index => Mock.Of<PlantList>())

.ToList();

        _currRepositoryMock.Setup(repo => repo.GetPlantLists()).ReturnsAsync(dbResponse);


        //Act
        var result = await _currServices.GetAffDistFinalOverallNumAssetResult(overAllNum, category);
        // Assert
        Assert.NotNull(result);
        Assert.IsType<List<AssetGroupedData>>(result);
    }
