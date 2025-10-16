[Fact]
public async Task PerformanceSummaryAffiliateAsync_ShouldReturnFullData_WithPopulatedKpisAndPlants()
{
    // Arrange
    var fixture = new Fixture();

    // Create sample request
    var request = fixture.Build<GetKpiPerformanceRequest>()
        .With(x => x.affiliateId, fixture.Create<int>())
        .With(x => x.performanceSummary, "MaintenanceExcellence") // meaningful summary
        .Create();

    // Sample plant hierarchy
    var plantDetails = fixture.CreateMany<GetCaseHierarchyResponse>(3).ToList();
    _configRepositoryMock.Setup(x => x.GetCaseHierarchyAsync(It.IsAny<GetCaseHierarchyRequest>()))
        .ReturnsAsync(plantDetails);

    // Sample affiliates
    var affiliateList = fixture.CreateMany<AffiliateList>(3).ToList();
    _currRepositoryMock.Setup(x => x.GetAffiliateLists())
        .ReturnsAsync(affiliateList);

    // Sample KPI formulas (must match the performanceSummary type)
    var kpiFormulas = fixture.Build<KpiFormulaTarget>()
        .With(x => x.name, "Energy Efficiency")
        .With(x => x.target, fixture.Create<decimal>())
        .CreateMany(2)
        .ToList();
    _performanceSumRepositoryMock
        .Setup(x => x.GetKpiFormulaTarget(It.IsAny<KpiFormulaTargetRequest>()))
        .ReturnsAsync(kpiFormulas);

    // Prepare actual data dictionaries for KPI calculations
    var actualData = kpiFormulas.ToDictionary(
        k => k.name.Replace(" ", "_"),
        k => plantDetails.ToDictionary(p => p.plantId ?? 0, _ => fixture.Create<decimal>())
    );

    var actualPlantData = kpiFormulas.ToDictionary(
        k => k.name.Replace(" ", "_"),
        k => plantDetails.ToDictionary(p => p.plantId ?? 0, _ => fixture.Create<decimal>())
    );

    // Use the actual service (no mocking of protected methods)
    var configurationMock = new Mock<IConfiguration>();
    var service = new PerformanceSummaryServices(
        _performanceSumRepositoryMock.Object,
        _currRepositoryMock.Object,
        configurationMock.Object,
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

    // Ensure the foreach loop added KPIs
    Assert.NotNull(result.kpis);
    Assert.True(result.kpis.Count > 0, "Kpi list should be populated");

    // Ensure plants are populated
    Assert.NotNull(result.plants);
    Assert.True(result.plants.Count > 0, "Plants list should be populated");

    // Ensure cost effectiveness calculation executed
    Assert.True(result.average >= 0, "Average should be >= 0");
    Assert.True(result.bestAffiliate >= 0, "Best affiliate percentage should be >= 0");
    Assert.False(string.IsNullOrWhiteSpace(result.bestAffiliateName), "Best affiliate name should not be empty");
}
