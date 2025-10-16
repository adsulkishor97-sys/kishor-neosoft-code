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

    var plantDetails = fixture.CreateMany<GetCaseHierarchyResponse>(3).ToList();
    _configRepositoryMock.Setup(x => x.GetCaseHierarchyAsync(It.IsAny<GetCaseHierarchyRequest>()))
        .ReturnsAsync(plantDetails);

    var affiliateList = fixture.CreateMany<AffiliateList>(3).ToList();
    _currRepositoryMock.Setup(x => x.GetAffiliateLists())
        .ReturnsAsync(affiliateList);

    var kpiFormulas = fixture.CreateMany<KpiFormulaTarget>(2).ToList();
    _performanceSumRepositoryMock
        .Setup(x => x.GetKpiFormulaTarget(It.IsAny<KpiFormulaTargetRequest>()))
        .ReturnsAsync(kpiFormulas);

    var _configurationMock = new Mock<IConfiguration>();

    // Use correct constructor with 4 parameters
    var service = new PerformanceSummaryServices(
        _performanceSumRepositoryMock.Object,
        _currRepositoryMock.Object,
        _configurationMock.Object,
        _configRepositoryMock.Object
    );

    // Act
    var result = await service.PerformanceSummaryAffiliateAsync(
        request,
        affiliateList[0].affiliateCode
    );

    // Assert
    Assert.NotNull(result);
    Assert.IsType<KpiPerformanceResponse>(result);
    Assert.NotNull(result.kpis);
    Assert.True(result.kpis.Count > 0);
    Assert.True(result.average >= 0);
    Assert.True(result.bestAffiliate >= 0);
    Assert.False(string.IsNullOrWhiteSpace(result.bestAffiliateName));
    Assert.NotNull(result.plants);
    Assert.True(result.plants.Count > 0);
}
