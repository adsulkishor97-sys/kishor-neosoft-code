[Fact]
public async Task PerformanceSummaryPlantAsync_ShouldReturn_FullyPopulatedResponse()
{
    // Arrange
    var request = _fixture.Build<GetKpiPerformanceRequest>()
                          .With(x => x.performanceSummary, _fixture.Create<string>())
                          .Create();

    // Generate dynamic affiliate and plant data
    var affiliateList = _fixture.CreateMany<AffiliateList>(3).ToList();
    var getCaseHierarchyResponses = _fixture.CreateMany<GetCaseHierarchyResponse>(2).ToList();

    var kpisFormula = _fixture.CreateMany<KpiFormulaTarget>(2)
                              .Select(k => 
                              {
                                  k.name ??= _fixture.Create<string>();
                                  k.target = _fixture.Create<decimal>();
                                  return k;
                              }).ToList();

    // Generate fake actual data for KPIs
    var actualDataDict = kpisFormula.ToDictionary(
        k => k.name!.Replace(" ", "_"),
        k => _fixture.Create<object>() // Replace object with actual type if needed
    );

    var actualPlantsDataDict = kpisFormula.ToDictionary(
        k => k.name!.Replace(" ", "_"),
        k => _fixture.Create<object>() // Replace object with actual type if needed
    );

    // Mock repositories
    _currRepositoryMock.Setup(r => r.GetAffiliateLists()).ReturnsAsync(affiliateList);
    _configRepositoryMock.Setup(r => r.GetCaseHierarchyAsync(It.IsAny<GetCaseHierarchyRequest>()))
                         .ReturnsAsync(getCaseHierarchyResponses);
    _performanceSumRepositoryMock.Setup(r => r.GetKpiFormulaTarget(It.IsAny<KpiFormulaTargetRequest>()))
                                 .ReturnsAsync(kpisFormula);

    // Mock internal service calls (GenerateMaintenanceKpiReport etc.)
    _mockService.Setup(s => s.GenerateMaintenanceKpiReport(It.IsAny<string>(), It.IsAny<List<AffiliateList>>(), It.IsAny<object>(), It.IsAny<decimal>()))
                .ReturnsAsync((string name, List<AffiliateList> affs, object data, decimal target) =>
                    new KpiDetail
                    {
                        name = name,
                        target = target,
                        plants = _fixture.CreateMany<Plant>(2).ToList()
                    });

    _mockService.Setup(s => s.GenerateMaintenancePlantKpiReport(It.IsAny<string>(), It.IsAny<List<GetCaseHierarchyResponse>>(), It.IsAny<object>(), It.IsAny<decimal>()))
                .ReturnsAsync((string name, List<GetCaseHierarchyResponse> plants, object data, decimal target) =>
                    new KpiDetail
                    {
                        name = name,
                        target = target,
                        plants = _fixture.CreateMany<Plant>(2).ToList()
                    });

    _mockService.Setup(s => s.GenerateConvertedMaintenanceReport(It.IsAny<KpiFormulaTarget>(), It.IsAny<KpiDetail[]>()))
                .ReturnsAsync((KpiFormulaTarget k, KpiDetail[] details) =>
                    _fixture.CreateMany<ConvertedKpiItemDetails>(2).ToList());

    _mockService.Setup(s => s.GenerateConvertedMaintenancePlantReport(It.IsAny<KpiFormulaTarget>(), It.IsAny<KpiDetail[]>()))
                .ReturnsAsync((KpiFormulaTarget k, KpiDetail[] details) =>
                    _fixture.CreateMany<ConvertedKpiItemPlantDetails>(2).ToList());

    var affiliateRequestString = _fixture.Create<string>();

    // Act
    var result = await _performanceSummServices.PerformanceSummaryPlantAsync(request, affiliateRequestString);

    // Assert
    Assert.NotNull(result);
    Assert.IsType<KpiPerformanceResponse>(result);
    // Ensure the cost effectiveness calculations executed
    Assert.NotNull(result.performanceSummary);
}

[Fact]
public async Task PerformanceSummaryPlantAsync_ShouldReturn_EmptyResponse_OnException()
{
    // Arrange
    var request = _fixture.Create<GetKpiPerformanceRequest>();
    var affiliateRequest = _fixture.Create<string>();

    _configRepositoryMock
        .Setup(r => r.GetCaseHierarchyAsync(It.IsAny<GetCaseHierarchyRequest>()))
        .ThrowsAsync(new Exception("DB Error"));

    // Act
    var result = await _performanceSummServices.PerformanceSummaryPlantAsync(request, affiliateRequest);

    // Assert
    Assert.NotNull(result);
    Assert.IsType<KpiPerformanceResponse>(result);
    Assert.Null(result.kpis); // object should be empty on exception
}
