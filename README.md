using Xunit;
using Moq;
using AutoFixture;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

public class PerformanceServiceTests
{
    private readonly Mock<IConfigRepository> _mockConfigRepo;
    private readonly Mock<ICurrentRepository> _mockCurrentRepo;
    private readonly Mock<PerformanceService> _serviceMock;
    private readonly Fixture _fixture;

    public PerformanceServiceTests()
    {
        _mockConfigRepo = new Mock<IConfigRepository>();
        _mockCurrentRepo = new Mock<ICurrentRepository>();
        _serviceMock = new Mock<PerformanceService>(_mockConfigRepo.Object, _mockCurrentRepo.Object) { CallBase = true };
        _fixture = new Fixture();
    }

    [Fact]
    public async Task PerformanceSummaryPlantAsync_ShouldCoverAllBranches_AutoFixture()
    {
        // Arrange
        var request = _fixture.Build<GetKpiPerformanceRequest>()
                              .With(x => x.performanceSummary, _fixture.Create<string>())
                              .Create();

        string affiliateRequest = _fixture.Create<string>();

        var plantDetails = _fixture.CreateMany<GetCaseHierarchyResponse>(2).ToList();
        var affiliates = _fixture.CreateMany<Affiliate>(2).ToList();
        var kpiFormulas = _fixture.CreateMany<KpiFormula>(2).ToList();

        var actualData = kpiFormulas.ToDictionary(
            k => k.name.Replace(" ", "_"),
            k => _fixture.CreateMany<ActualData>(2).ToList()
        );

        var actualPlantData = kpiFormulas.ToDictionary(
            k => k.name.Replace(" ", "_"),
            k => _fixture.CreateMany<ActualData>(2).ToList()
        );

        _mockConfigRepo.Setup(x => x.GetCaseHierarchyAsync(It.IsAny<GetCaseHierarchyRequest>()))
            .ReturnsAsync(plantDetails);

        _mockCurrentRepo.Setup(x => x.GetAffiliateLists())
            .ReturnsAsync(affiliates);

        _serviceMock.Setup(x => x.GetKpiFormulas(It.IsAny<KpiFormulaTargetRequest>()))
            .Returns(kpiFormulas);

        _serviceMock.Setup(x => x.GetPerformanceSummaryActualDataNew(It.IsAny<GetKpiPerformanceRequest>(), It.IsAny<string>()))
            .Returns(actualData);

        _serviceMock.Setup(x => x.GetPerformanceSummaryAffiliateActualData(It.IsAny<GetKpiPerformanceRequest>(), It.IsAny<string>(), It.IsAny<List<GetCaseHierarchyResponse>>()))
            .Returns(actualPlantData);

        _serviceMock.Setup(x => x.GenerateMaintenanceKpiReport(It.IsAny<string>(), It.IsAny<List<Affiliate>>(), It.IsAny<List<ActualData>>(), It.IsAny<decimal>()))
            .ReturnsAsync(_fixture.Create<KpiDetail>());

        _serviceMock.Setup(x => x.GenerateMaintenancePlantKpiReport(It.IsAny<string>(), It.IsAny<List<GetCaseHierarchyResponse>>(), It.IsAny<List<ActualData>>(), It.IsAny<decimal>()))
            .ReturnsAsync(_fixture.Create<KpiDetail>());

        _serviceMock.Setup(x => x.GenerateConvertedMaintenanceReport(It.IsAny<KpiFormula>(), It.IsAny<KpiDetail[]>()))
            .ReturnsAsync(_fixture.CreateMany<ConvertedKpiItemDetails>(2).ToList());

        _serviceMock.Setup(x => x.GenerateConvertedMaintenancePlantReport(It.IsAny<KpiFormula>(), It.IsAny<KpiDetail>()))
            .ReturnsAsync(_fixture.CreateMany<ConvertedKpiItemPlantDetails>(2).ToList());

        // Act
        var result = await _serviceMock.Object.PerformanceSummaryPlantAsync(request, affiliateRequest);

        // Assert
        Assert.NotNull(result);
        Assert.Equal(request.performanceSummary, result.performanceSummary);
    }

    [Fact]
    public async Task PerformanceSummaryPlantAsync_ShouldHandleException_AutoFixture()
    {
        // Arrange
        var request = _fixture.Create<GetKpiPerformanceRequest>();
        string affiliateRequest = _fixture.Create<string>();

        _mockConfigRepo.Setup(x => x.GetCaseHierarchyAsync(It.IsAny<GetCaseHierarchyRequest>()))
            .ThrowsAsync(new System.Exception("Forced exception"));

        // Act
        var result = await _serviceMock.Object.PerformanceSummaryPlantAsync(request, affiliateRequest);

        // Assert
        Assert.NotNull(result); // Returns empty KpiPerformanceResponse
    }
}
