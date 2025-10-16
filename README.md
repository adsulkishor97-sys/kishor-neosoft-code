[Fact]
public async Task PerformanceSummaryAffiliateAsync_ShouldCoverLoopAndCostEffectivenessLogic_UsingAutoFixture()
{
    // Arrange
    var fixture = new Fixture();

    // Create input parameters dynamically
    var request = fixture.Build<GetKpiPerformanceRequest>()
        .With(x => x.performanceSummary, fixture.Create<string>())
        .With(x => x.affiliateId, fixture.Create<int>())
        .Create();

    var affiliateRequest = fixture.Create<string>();

    // Mock repositories
    var mockConfigRepo = new Mock<IConfigRepository>();
    var mockCurrentRepo = new Mock<ICurrentRepository>();

    var affiliatesRes = fixture.CreateMany<AffiliateList>(fixture.Create<int>() % 3 + 2).ToList();
    var plantDetails = fixture.CreateMany<GetCaseHierarchyResponse>(fixture.Create<int>() % 3 + 2).ToList();

    mockConfigRepo.Setup(x => x.GetCaseHierarchyAsync(It.IsAny<GetCaseHierarchyRequest>()))
        .ReturnsAsync(plantDetails);

    mockCurrentRepo.Setup(x => x.GetAffiliateLists())
        .ReturnsAsync(affiliatesRes);

    // Create partial mock of service
    var service = new Mock<PerformanceSummaryServices>(mockConfigRepo.Object, mockCurrentRepo.Object) { CallBase = true };

    // Mock KpiFormulaTarget list
    var kpiFormulas = fixture.CreateMany<KpiFormulaTarget>(1)
        .Select(f => fixture.Build<KpiFormulaTarget>()
            .With(x => x.name, fixture.Create<string>())
            .With(x => x.target, Math.Round(fixture.Create<decimal>() + 1, 2))
            .Create())
        .ToList();

    service.Protected()
        .Setup<List<KpiFormulaTarget>>("GetKpiFormulas", ItExpr.IsAny<KpiFormulaTargetRequest>())
        .Returns(kpiFormulas);

    // Create dynamic actual data dictionaries using fixture
    var actualData = new Dictionary<string, Dictionary<int, decimal>>();
    var actualPlantData = new Dictionary<string, Dictionary<int, decimal>>();

    foreach (var kpi in kpiFormulas)
    {
        var key = kpi.name.Replace(" ", "_");
        actualData[key] = plantDetails.ToDictionary(p => p.plantId ?? fixture.Create<int>(), _ => fixture.Create<decimal>());
        actualPlantData[key] = plantDetails.ToDictionary(p => p.plantId ?? fixture.Create<int>(), _ => fixture.Create<decimal>());
    }

    service.Protected()
        .Setup<Dictionary<string, Dictionary<int, decimal>>>("GetPerformanceSummaryActualDataNew",
            ItExpr.IsAny<GetKpiPerformanceRequest>(), ItExpr.IsAny<string>())
        .Returns(actualData);

    service.Protected()
        .Setup<Dictionary<string, Dictionary<int, decimal>>>("GetPerformanceSummaryAffiliateActualData",
            ItExpr.IsAny<GetKpiPerformanceRequest>(), ItExpr.IsAny<string>(), ItExpr.IsAny<List<GetCaseHierarchyResponse>>())
        .Returns(actualPlantData);

    // Mock report generation methods (use fixture-generated data)
    var report = fixture.Create<KpiDetail>();
    var plantReport = fixture.Create<KpiDetail>();
    var convertedReport = fixture.CreateMany<ConvertedKpiItemDetails>(fixture.Create<int>() % 3 + 2).ToList();
    var convertedPlantReport = fixture.CreateMany<ConvertedKpiItemPlantDetails>(fixture.Create<int>() % 3 + 2).ToList();

    service.Protected()
        .Setup<Task<KpiDetail>>("GenerateMaintenanceKpiReport",
            ItExpr.IsAny<string>(),
            ItExpr.IsAny<List<AffiliateList>>(),
            ItExpr.IsAny<Dictionary<int, decimal>>(),
            ItExpr.IsAny<decimal>())
        .ReturnsAsync(report);

    service.Protected()
        .Setup<Task<KpiDetail>>("GenerateMaintenancePlantKpiReport",
            ItExpr.IsAny<string>(),
            ItExpr.IsAny<List<GetCaseHierarchyResponse>>(),
            ItExpr.IsAny<Dictionary<int, decimal>>(),
            ItExpr.IsAny<decimal>())
        .ReturnsAsync(plantReport);

    service.Protected()
        .Setup<Task<List<ConvertedKpiItemDetails>>>("GenerateConvertedMaintenanceReport",
            ItExpr.IsAny<KpiFormulaTarget>(), ItExpr.IsAny<KpiDetail>())
        .ReturnsAsync(convertedReport);

    service.Protected()
        .Setup<Task<List<ConvertedKpiItemPlantDetails>>>("GenerateConvertedMaintenancePlantReport",
            ItExpr.IsAny<KpiFormulaTarget>(), ItExpr.IsAny<KpiDetail>())
        .ReturnsAsync(convertedPlantReport);

    // Act
    var result = await service.Object.PerformanceSummaryAffiliateAsync(request, affiliateRequest);

    // Assert — ensures loop & cost-effectiveness logic executed
    Assert.NotNull(result);
    Assert.NotEmpty(result.kpis);
    Assert.True(result.average >= 0);
    Assert.True(result.bestAffiliate >= 0);
    Assert.False(string.IsNullOrWhiteSpace(result.bestAffiliateName));
    Assert.True(result.target >= 0);
    Assert.NotEmpty(result.plants);

    // Verify key methods were called inside the foreach loop
    service.Protected().Verify<Task<KpiDetail>>(
        "GenerateMaintenanceKpiReport",
        Times.AtLeastOnce(),
        ItExpr.IsAny<string>(),
        ItExpr.IsAny<List<AffiliateList>>(),
        ItExpr.IsAny<Dictionary<int, decimal>>(),
        ItExpr.IsAny<decimal>());

    service.Protected().Verify<Task<KpiDetail>>(
        "GenerateMaintenancePlantKpiReport",
        Times.AtLeastOnce(),
        ItExpr.IsAny<string>(),
        ItExpr.IsAny<List<GetCaseHierarchyResponse>>(),
        ItExpr.IsAny<Dictionary<int, decimal>>(),
        ItExpr.IsAny<decimal>());
}
