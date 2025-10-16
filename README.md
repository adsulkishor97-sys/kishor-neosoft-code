[Fact]
public async Task PerformanceSummaryAffiliateAsync_ShouldReturnCalculatedValues_WhenValidDataProvided()
{
    // Arrange
    var fixture = new Fixture();

    var request = fixture.Build<GetKpiPerformanceRequest>()
        .With(x => x.performanceSummary, "MaintenanceExcellence")
        .With(x => x.affiliateId, fixture.Create<int>())
        .Create();

    var affiliateRequest = fixture.Create<string>();

    var mockConfigRepo = new Mock<IConfigRepository>();
    var mockCurrentRepo = new Mock<ICurrentRepository>();

    // Mock affiliate and plant data
    var affiliatesRes = fixture.CreateMany<AffiliateList>(3).ToList();
    var plantDetails = fixture.CreateMany<GetCaseHierarchyResponse>(2).ToList();

    mockCurrentRepo.Setup(x => x.GetAffiliateLists())
        .ReturnsAsync(affiliatesRes);

    mockConfigRepo.Setup(x => x.GetCaseHierarchyAsync(It.IsAny<GetCaseHierarchyRequest>()))
        .ReturnsAsync(plantDetails);

    // Mock dependent private methods using partial mock of service
    var service = new Mock<MyServiceUnderTest>(mockConfigRepo.Object, mockCurrentRepo.Object) { CallBase = true };

    // Mock GetKpiFormulas
    var kpiFormulaList = fixture.Build<KpiFormulaTarget>()
        .With(k => k.name, "KPI_1")
        .With(k => k.target, 100m)
        .CreateMany(1).ToList();

    service.Protected()
        .Setup<List<KpiFormulaTarget>>("GetKpiFormulas", ItExpr.IsAny<KpiFormulaTargetRequest>())
        .Returns(kpiFormulaList);

    // Mock GetPerformanceSummaryActualDataNew
    var actualData = new Dictionary<string, List<KpiNumeratorDenominatorPerformanceSummary>>
    {
        { "KPI_1", fixture.CreateMany<KpiNumeratorDenominatorPerformanceSummary>(3).ToList() }
    };

    service.Protected()
        .Setup<Dictionary<string, List<KpiNumeratorDenominatorPerformanceSummary>>>("GetPerformanceSummaryActualDataNew",
            ItExpr.IsAny<GetKpiPerformanceRequest>(), ItExpr.IsAny<string>())
        .Returns(actualData);

    // Mock GetPerformanceSummaryAffiliateActualData
    var actualPlantData = new Dictionary<string, List<KpiNumeratorDenominatorPerformanceSummary>>
    {
        { "KPI_1", fixture.CreateMany<KpiNumeratorDenominatorPerformanceSummary>(3).ToList() }
    };

    service.Protected()
        .Setup<Dictionary<string, List<KpiNumeratorDenominatorPerformanceSummary>>>("GetPerformanceSummaryAffiliateActualData",
            ItExpr.IsAny<GetKpiPerformanceRequest>(), ItExpr.IsAny<string>(), ItExpr.IsAny<List<GetCaseHierarchyResponse>>())
        .Returns(actualPlantData);

    // Mock report generation methods
    var report = fixture.Create<KpiDetail>();
    var plantReport = fixture.Create<KpiDetail>();
    var convertedReport = fixture.CreateMany<ConvertedKpiItemDetails>(3).ToList();
    var convertedPlantReport = fixture.CreateMany<ConvertedKpiItemPlantDetails>(2).ToList();

    service.Protected().Setup<Task<KpiDetail>>("GenerateMaintenanceKpiReport", ItExpr.IsAny<string>(),
        ItExpr.IsAny<List<AffiliateList>>(), ItExpr.IsAny<List<KpiNumeratorDenominatorPerformanceSummary>>(), ItExpr.IsAny<decimal>())
        .ReturnsAsync(report);

    service.Protected().Setup<Task<KpiDetail>>("GenerateMaintenancePlantKpiReport", ItExpr.IsAny<string>(),
        ItExpr.IsAny<List<GetCaseHierarchyResponse>>(), ItExpr.IsAny<List<KpiNumeratorDenominatorPerformanceSummary>>(), ItExpr.IsAny<decimal>())
        .ReturnsAsync(plantReport);

    service.Protected().Setup<Task<List<ConvertedKpiItemDetails>>>("GenerateConvertedMaintenanceReport",
        ItExpr.IsAny<KpiFormulaTarget>(), ItExpr.IsAny<KpiDetail>())
        .ReturnsAsync(convertedReport);

    service.Protected().Setup<Task<List<ConvertedKpiItemPlantDetails>>>("GenerateConvertedMaintenancePlantReport",
        ItExpr.IsAny<KpiFormulaTarget>(), ItExpr.IsAny<KpiDetail>())
        .ReturnsAsync(convertedPlantReport);

    // Act
    var result = await service.Object.PerformanceSummaryAffiliateAsync(request, affiliateRequest);

    // Assert
    Assert.NotNull(result);
    Assert.NotEmpty(result.kpis);
    Assert.True(result.average > 0);
    Assert.NotNull(result.bestAffiliateName);
    Assert.True(result.target > 0);
    Assert.NotEmpty(result.plants);

    // Verify key methods executed (ensures foreach loop was covered)
    service.Protected().Verify<Task<KpiDetail>>(
        "GenerateMaintenanceKpiReport", Times.Once(),
        ItExpr.IsAny<string>(), ItExpr.IsAny<List<AffiliateList>>(),
        ItExpr.IsAny<List<KpiNumeratorDenominatorPerformanceSummary>>(), ItExpr.IsAny<decimal>());

    service.Protected().Verify<Task<List<ConvertedKpiItemDetails>>>(
        "GenerateConvertedMaintenanceReport", Times.Once(),
        ItExpr.IsAny<KpiFormulaTarget>(), ItExpr.IsAny<KpiDetail>());
}
