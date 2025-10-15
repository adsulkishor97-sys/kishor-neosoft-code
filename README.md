[Fact]
public async Task PerformanceSummaryPlantAsync_ShouldExecuteFullFlow_AndReturnComputedResult()
{
    // Arrange
    var fixture = new Fixture();

    // Mock Repositories
    var configRepoMock = new Mock<IConfigRepository>();
    var currentRepoMock = new Mock<ICurrentRepository>();

    // Create sample request dynamically
    var request = fixture.Build<GetKpiPerformanceRequest>()
        .With(x => x.affiliateId, fixture.Create<int>())
        .With(x => x.performanceSummary, fixture.Create<string>())
        .With(x => x.plantId, fixture.Create<int>())
        .Create();

    // Generate mock data for repository calls
    var plantDetails = fixture.CreateMany<GetCaseHierarchyResponse>(3).ToList();
    var affiliateList = fixture.CreateMany<AffiliateList>(3).ToList();

    configRepoMock.Setup(x => x.GetCaseHierarchyAsync(It.IsAny<GetCaseHierarchyRequest>()))
        .ReturnsAsync(plantDetails);

    currentRepoMock.Setup(x => x.GetAffiliateLists())
        .ReturnsAsync(affiliateList);

    // Create service (you may have to make internal helpers virtual for Moq)
    var service = new Mock<PerformanceSummaryServices>(configRepoMock.Object, currentRepoMock.Object)
    { CallBase = true };

    // Mock GetKpiFormulas() to return valid KPIs
    var kpiFormulas = fixture.Build<KpiFormulaTarget>()
        .With(x => x.name, "Energy Efficiency")
        .With(x => x.target, 100m)
        .CreateMany(2)
        .ToList();

    service.Protected()
        .Setup<List<KpiFormulaTarget>>("GetKpiFormulas", ItExpr.IsAny<KpiFormulaTargetRequest>())
        .Returns(kpiFormulas);

    // Prepare fake actual data dictionaries
    var actualData = new Dictionary<string, Dictionary<int, decimal>>
    {
        ["Energy_Efficiency"] = plantDetails.ToDictionary(p => p.plantId ?? 0, _ => fixture.Create<decimal>())
    };

    var actualPlantData = new Dictionary<string, Dictionary<int, decimal>>
    {
        ["Energy_Efficiency"] = plantDetails.ToDictionary(p => p.plantId ?? 0, _ => fixture.Create<decimal>())
    };

    // Mock actual data methods
    service.Protected()
        .Setup<Dictionary<string, Dictionary<int, decimal>>>("GetPerformanceSummaryActualDataNew",
            ItExpr.IsAny<GetKpiPerformanceRequest>(), ItExpr.IsAny<string>())
        .Returns(actualData);

    service.Protected()
        .Setup<Dictionary<string, Dictionary<int, decimal>>>("GetPerformanceSummaryAffiliateActualData",
            ItExpr.IsAny<GetKpiPerformanceRequest>(), ItExpr.IsAny<string>(), ItExpr.IsAny<List<GetCaseHierarchyResponse>>())
        .Returns(actualPlantData);

    // Mock reports
    var kpiReport = fixture.Build<KpiDetail>()
        .With(x => x.kpi, "Energy Efficiency")
        .With(x => x.plants, fixture.CreateMany<Plant>(3).ToList())
        .Create();

    var plantKpiReport = fixture.Build<KpiDetail>()
        .With(x => x.kpi, "Energy Efficiency")
        .With(x => x.plants, fixture.CreateMany<Plant>(3).ToList())
        .Create();

    service.Protected()
        .Setup<Task<KpiDetail>>("GenerateMaintenanceKpiReport", 
            ItExpr.IsAny<string>(), ItExpr.IsAny<List<AffiliateList>>(), 
            ItExpr.IsAny<Dictionary<int, decimal>>(), ItExpr.IsAny<decimal>())
        .ReturnsAsync(kpiReport);

    service.Protected()
        .Setup<Task<KpiDetail>>("GenerateMaintenancePlantKpiReport",
            ItExpr.IsAny<string>(), ItExpr.IsAny<List<GetCaseHierarchyResponse>>(),
            ItExpr.IsAny<Dictionary<int, decimal>>(), ItExpr.IsAny<decimal>())
        .ReturnsAsync(plantKpiReport);

    // Mock converted reports
    service.Protected()
        .Setup<Task<List<ConvertedKpiItemDetails>>>("GenerateConvertedMaintenanceReport",
            ItExpr.IsAny<KpiFormulaTarget>(), ItExpr.IsAny<KpiDetail>())
        .ReturnsAsync(fixture.CreateMany<ConvertedKpiItemDetails>(3).ToList());

    service.Protected()
        .Setup<Task<List<ConvertedKpiItemPlantDetails>>>("GenerateConvertedMaintenancePlantReport",
            ItExpr.IsAny<KpiFormulaTarget>(), ItExpr.IsAny<KpiDetail>())
        .ReturnsAsync(fixture.CreateMany<ConvertedKpiItemPlantDetails>(3).ToList());

    // Act
    var result = await service.Object.PerformanceSummaryPlantAsync(request, fixture.Create<string>());

    // Assert
    Assert.NotNull(result);
    Assert.True(result.average >= 0);
    Assert.True(result.bestPlant >= 0);
    Assert.False(string.IsNullOrWhiteSpace(result.bestPlantName));
    Assert.NotNull(result.performanceSummary);
}
