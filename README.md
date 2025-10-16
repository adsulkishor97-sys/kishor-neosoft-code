public class PerformanceSummaryServicesTests
{
    private readonly Fixture _fixture;
    private readonly Mock<IConfigRepository> _configRepositoryMock;
    private readonly Mock<ICurrentRepository> _currRepositoryMock;
    private readonly Mock<IPerformanceSummaryRepository> _performanceSumRepositoryMock;

    public PerformanceSummaryServicesTests()
    {
        _fixture = new Fixture();
        _configRepositoryMock = new Mock<IConfigRepository>();
        _currRepositoryMock = new Mock<ICurrentRepository>();
        _performanceSumRepositoryMock = new Mock<IPerformanceSummaryRepository>();
    }

    [Fact]
    public async Task PerformanceSummaryAffiliateAsync_ShouldCoverLoopAndCostEffectivenessLogic()
    {
        // Arrange
        var request = _fixture.Create<GetKpiPerformanceRequest>();
        var affiliateRequest = _fixture.Create<string>();

        // Mock repositories
        var affiliateList = _fixture.CreateMany<AffiliateList>(3).ToList();
        var plantDetails = _fixture.CreateMany<GetCaseHierarchyResponse>(3).ToList();

        _currRepositoryMock.Setup(x => x.GetAffiliateLists()).ReturnsAsync(affiliateList);
        _configRepositoryMock.Setup(x => x.GetCaseHierarchyAsync(It.IsAny<GetCaseHierarchyRequest>())).ReturnsAsync(plantDetails);

        // Mock formulas
        var kpiFormulas = _fixture.CreateMany<KpiFormulaTarget>(2)
            .Select(f => _fixture.Build<KpiFormulaTarget>()
                .With(x => x.name, _fixture.Create<string>())
                .With(x => x.target, Math.Round(_fixture.Create<decimal>() + 1, 2))
                .Create())
            .ToList();

        _performanceSumRepositoryMock.Setup(x => x.GetKpiFormulaTarget(It.IsAny<KpiFormulaTargetRequest>()))
            .ReturnsAsync(kpiFormulas);

        // Mock Generate methods
        var kpiDetail = _fixture.Create<KpiDetail>();
        var plantKpiDetail = _fixture.Create<KpiDetail>();
        var converted = _fixture.CreateMany<ConvertedKpiItemDetails>(2).ToList();
        var convertedPlant = _fixture.CreateMany<ConvertedKpiItemPlantDetails>(2).ToList();

        var serviceMock = new Mock<PerformanceSummaryServices>(_configRepositoryMock.Object, _currRepositoryMock.Object) { CallBase = true };

        serviceMock.Protected()
            .Setup<Task<KpiDetail>>("GenerateMaintenanceKpiReport", ItExpr.IsAny<string>(), ItExpr.IsAny<List<AffiliateList>>(),
                ItExpr.IsAny<Dictionary<int, decimal>>(), ItExpr.IsAny<decimal>())
            .ReturnsAsync(kpiDetail);

        serviceMock.Protected()
            .Setup<Task<KpiDetail>>("GenerateMaintenancePlantKpiReport", ItExpr.IsAny<string>(), ItExpr.IsAny<List<GetCaseHierarchyResponse>>(),
                ItExpr.IsAny<Dictionary<int, decimal>>(), ItExpr.IsAny<decimal>())
            .ReturnsAsync(plantKpiDetail);

        serviceMock.Protected()
            .Setup<Task<List<ConvertedKpiItemDetails>>>("GenerateConvertedMaintenanceReport", ItExpr.IsAny<KpiFormulaTarget>(), ItExpr.IsAny<KpiDetail>())
            .ReturnsAsync(converted);

        serviceMock.Protected()
            .Setup<Task<List<ConvertedKpiItemPlantDetails>>>("GenerateConvertedMaintenancePlantReport", ItExpr.IsAny<KpiFormulaTarget>(), ItExpr.IsAny<KpiDetail>())
            .ReturnsAsync(convertedPlant);

        // Act
        var result = await serviceMock.Object.PerformanceSummaryAffiliateAsync(request, affiliateRequest);

        // Assert
        Assert.NotNull(result);
        Assert.NotEmpty(result.kpis); // reports added
        Assert.True(result.average >= 0); // cost-effectiveness computed
        Assert.True(result.bestAffiliate >= 0);
        Assert.False(string.IsNullOrWhiteSpace(result.bestAffiliateName));
        Assert.NotEmpty(result.plants); // plant report merged

        // Verify protected GenerateMaintenanceKpiReport called
        serviceMock.Protected().Verify<Task<KpiDetail>>("GenerateMaintenanceKpiReport", Times.AtLeastOnce(),
            ItExpr.IsAny<string>(), ItExpr.IsAny<List<AffiliateList>>(), ItExpr.IsAny<Dictionary<int, decimal>>(), ItExpr.IsAny<decimal>());
    }
}
 System.ArgumentException : No protected method PerformanceSummaryServices.GenerateMaintenanceKpiReport found whose signature is compatible with the provided arguments (string, List<AffiliateList>, Dictionary<int, decimal>, decimal).
