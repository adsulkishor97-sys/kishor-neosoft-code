[Fact]
public async Task PerformanceSummaryAffiliateAsync_ShouldExecuteFullFlow_WithActualProtectedMethods()
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

    // Sample KPI formulas
    var kpiFormulas = fixture.CreateMany<KpiFormulaTarget>(2).ToList();
    _performanceSumRepositoryMock
        .Setup(x => x.GetKpiFormulaTarget(It.IsAny<KpiFormulaTargetRequest>()))
        .ReturnsAsync(kpiFormulas);

    // Use the actual service class (no mocking of protected methods)
    var service = new PerformanceSummaryServices(
        _configRepositoryMock.Object,
        _currRepositoryMock.Object,
        _performanceSumRepositoryMock.Object
    );

    // Act
    var result = await service.PerformanceSummaryAffiliateAsync(
        request,
        affiliateList[0].affiliateCode
    );

    // Assert
    Assert.NotNull(result);
    Assert.IsType<KpiPerformanceResponse>(result);

    // Ensure protected methods actually ran and populated kpis & plants
    Assert.NotNull(result.kpis);
    Assert.True(result.kpis.Count > 0, "Kpi list should be populated");

    Assert.True(result.average >= 0, "Average percentage should be calculated");
    Assert.True(result.bestAffiliate >= 0, "Best affiliate should be calculated");
    Assert.False(string.IsNullOrWhiteSpace(result.bestAffiliateName), "Best affiliate name should be set");

    Assert.NotNull(result.plants);
    Assert.True(result.plants.Count > 0, "Plants list should be populated after cost effectiveness calculation");
}
