[Fact]
public async Task PerformanceSummaryAffiliateAsync_ShouldReturnData_WithFullFlow()
{
    // Arrange
    var fixture = new Fixture();

    // Sample request
    var request = fixture.Build<GetKpiPerformanceRequest>()
        .With(x => x.affiliateId, fixture.Create<int>())
        .With(x => x.performanceSummary, fixture.Create<string>())
        .Create();

    // Sample plant hierarchy
    var plantDetails = fixture.CreateMany<GetCaseHierarchyResponse>(3).ToList();
    _configRepositoryMock.Setup(x => x.GetCaseHierarchyAsync(It.IsAny<GetCaseHierarchyRequest>()))
        .ReturnsAsync(plantDetails);

    // Sample affiliates
    var affiliateList = fixture.CreateMany<AffiliateList>(3).ToList();
    _currRepositoryMock.Setup(x => x.GetAffiliateLists())
        .ReturnsAsync(affiliateList);

    // Sample KPI formulas returned from repository
    var kpiFormulas = fixture.CreateMany<KpiFormulaTarget>(2).ToList();
    _performanceSumRepositoryMock
        .Setup(x => x.GetKpiFormulaTarget(It.IsAny<KpiFormulaTargetRequest>()))
        .ReturnsAsync(kpiFormulas);

    // Act
    var result = await _performanceSummServices.PerformanceSummaryAffiliateAsync(
        request,
        affiliateList[0].affiliateCode.ToString() // pass any affiliate code
    );

    // Assert
    Assert.NotNull(result);
    Assert.IsType<KpiPerformanceResponse>(result);

    // Ensure private logic that modifies kpis and plants runs
    Assert.NotNull(result.kpis); // The foreach adding kpis
    Assert.True(result.kpis.Count > 0);

    // Ensure cost effectiveness calculation executes
    Assert.True(result.average >= 0 || result.average == 0);
    Assert.True(result.bestAffiliate >= 0 || result.bestAffiliate == 0);
    Assert.False(string.IsNullOrWhiteSpace(result.bestAffiliateName));
    Assert.NotNull(result.plants);
    Assert.True(result.plants.Count > 0);
}
 Assert.NotNull() Failure: Value is null
