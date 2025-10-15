[Fact]
public async Task PerformanceSummaryPlantAsync_ShouldCalculateMetricsAndReturnExpectedResponse()
{
    // Arrange
    var fixture = new Fixture();

    // Mock repositories
    var configRepoMock = new Mock<IConfigRepository>();
    var currentRepoMock = new Mock<ICurrentRepository>();

    // Create service under test
    var service = new PerformanceSummaryServices(configRepoMock.Object, currentRepoMock.Object);

    // AutoFixture data
    var request = fixture.Build<GetKpiPerformanceRequest>()
        .With(x => x.affiliateId, fixture.Create<int>())
        .With(x => x.performanceSummary, fixture.Create<string>())
        .With(x => x.plantId, fixture.Create<int>())
        .Create();

    var affiliateList = fixture.CreateMany<AffiliateList>(3).ToList();
    var plantDetails = fixture.CreateMany<GetCaseHierarchyResponse>(3).ToList();
    var kpiFormulaList = fixture.Build<KpiFormulaTarget>()
        .With(x => x.name, fixture.Create<string>())
        .With(x => x.target, fixture.Create<decimal>())
        .CreateMany(2)
        .ToList();

    // Simulate BigData results
    var actualData = kpiFormulaList.ToDictionary(
        k => k.name!.Replace(" ", "_"),
        k => fixture.Create<Dictionary<int, decimal>>()
    );

    var actualPlantsData = kpiFormulaList.ToDictionary(
        k => k.name!.Replace(" ", "_"),
        k => fixture.Create<Dictionary<int, decimal>>()
    );

    // Mock repository responses
    configRepoMock.Setup(x => x.GetCaseHierarchyAsync(It.IsAny<GetCaseHierarchyRequest>()))
        .ReturnsAsync(plantDetails);

    currentRepoMock.Setup(x => x.GetAffiliateLists())
        .ReturnsAsync(affiliateList);

    // Partial mocking for internal methods using reflection OR exposing internal methods via InternalsVisibleTo
    var privateMethods = typeof(PerformanceSummaryServices)
        .GetMethods(BindingFlags.NonPublic | BindingFlags.Instance)
        .Select(m => m.Name)
        .ToList();

    // Act
    var result = await service.PerformanceSummaryPlantAsync(request, fixture.Create<string>());

    // Assert
    Assert.NotNull(result);
    Assert.True(result.average >= 0);
    Assert.NotNull(result.bestPlantName);
    Assert.True(result.bestPlant >= 0);
}
