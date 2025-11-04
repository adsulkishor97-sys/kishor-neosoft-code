using Xunit;
using Moq;
using AutoFixture;
using System;
using System.Linq;
using System.Threading.Tasks;
using System.Collections.Generic;
using Microsoft.Extensions.Configuration;

public class PerformanceSummaryPlantAsyncTests
{
    private readonly IFixture _fixture;
    private readonly Mock<ICurrentRepository> _currRepositoryMock;
    private readonly Mock<IConfigRepository> _configRepositoryMock;
    private readonly Mock<IPerformanceSummaryRepository> _performanceSumRepositoryMock;
    private readonly PerformanceSummaryServices _performanceSummServices;

    public PerformanceSummaryPlantAsyncTests()
    {
        _fixture = new Fixture();
        _currRepositoryMock = new Mock<ICurrentRepository>();
        _configRepositoryMock = new Mock<IConfigRepository>();
        _performanceSumRepositoryMock = new Mock<IPerformanceSummaryRepository>();

        var mockConfig = new Mock<IConfiguration>();

        _performanceSummServices = new PerformanceSummaryServices(
            _currRepositoryMock.Object,
            _configRepositoryMock.Object,
            _performanceSumRepositoryMock.Object,
            mockConfig.Object
        );
    }

    // ✅ 1. Main Happy Path — Covers all loops, groups, and calculations
    [Fact]
    public async Task PerformanceSummaryPlantAsync_ShouldReturnExpectedResult_WhenDependenciesReturnValidData()
    {
        // Arrange
        var request = _fixture.Build<GetKpiPerformanceRequest>()
                              .With(x => x.affiliateId, _fixture.Create<int>())
                              .With(x => x.plantId, _fixture.Create<int>())
                              .With(x => x.performanceSummary, _fixture.Create<string>())
                              .Create();

        var affiliateList = _fixture.Build<AffiliateList>()
                                    .With(a => a.affiliateId, request.affiliateId)
                                    .CreateMany(3).ToList();

        var getCaseHierarchyResponses = _fixture.Build<GetCaseHierarchyResponse>()
                                                .With(x => x.plantId, request.plantId)
                                                .With(x => x.plantName, _fixture.Create<string>())
                                                .CreateMany(3).ToList();

        var kpiFormulaTargets = _fixture.Build<KpiFormulaTarget>()
                                        .With(x => x.kpi, _fixture.Create<string>())
                                        .With(x => x.name, _fixture.Create<string>())
                                        .With(x => x.target, _fixture.Create<decimal>())
                                        .CreateMany(2).ToList();

        // Mock repository methods
        _currRepositoryMock.Setup(r => r.GetAffiliateLists()).ReturnsAsync(affiliateList);
        _configRepositoryMock.Setup(r => r.GetCaseHierarchyAsync(It.IsAny<GetCaseHierarchyRequest>()))
                             .ReturnsAsync(getCaseHierarchyResponses);

        // Mock kpi formula data
        _performanceSumRepositoryMock.Setup(r => r.GetKpiFormulaTarget(It.IsAny<KpiFormulaTargetRequest>()))
                                     .ReturnsAsync(kpiFormulaTargets);

        // Mock internal helper methods inside service (to isolate DB logic)
        var serviceMock = new Mock<PerformanceSummaryServices>(
            _currRepositoryMock.Object,
            _configRepositoryMock.Object,
            _performanceSumRepositoryMock.Object,
            new Mock<IConfiguration>().Object
        ) { CallBase = true };

        // mock data for internal processing
        var kpiName = kpiFormulaTargets.First().name!.Replace(" ", "_");
        var fakeActualData = new Dictionary<string, List<KpiDetail>>
        {
            { kpiName, _fixture.CreateMany<KpiDetail>(2).ToList() }
        };
        var fakeActualPlantsData = new Dictionary<string, List<KpiDetail>>
        {
            { kpiName, _fixture.CreateMany<KpiDetail>(2).ToList() }
        };

        serviceMock.Setup(s => s.GetKpiFormulas(It.IsAny<KpiFormulaTargetRequest>()))
                   .Returns(kpiFormulaTargets);

        serviceMock.Setup(s => s.GetPerformanceSummaryActualDataNew(It.IsAny<GetKpiPerformanceRequest>(), It.IsAny<string>()))
                   .Returns(fakeActualData);

        serviceMock.Setup(s => s.GetPerformanceSummaryAffiliateActualData(It.IsAny<GetKpiPerformanceRequest>(), It.IsAny<string>(), It.IsAny<List<GetCaseHierarchyResponse>>()))
                   .Returns(fakeActualPlantsData);

        serviceMock.Setup(s => s.GenerateMaintenanceKpiReport(It.IsAny<string>(), It.IsAny<List<AffiliateList>>(), It.IsAny<List<KpiDetail>>(), It.IsAny<decimal>()))
                   .ReturnsAsync(_fixture.Create<KpiDetail>());
        serviceMock.Setup(s => s.GenerateMaintenancePlantKpiReport(It.IsAny<string>(), It.IsAny<List<GetCaseHierarchyResponse>>(), It.IsAny<List<KpiDetail>>(), It.IsAny<decimal>()))
                   .ReturnsAsync(_fixture.Create<KpiDetail>());
        serviceMock.Setup(s => s.GenerateConvertedMaintenanceReport(It.IsAny<KpiFormulaTarget>(), It.IsAny<KpiDetail>()))
                   .ReturnsAsync(_fixture.CreateMany<ConvertedKpiItemDetails>(3).ToList());
        serviceMock.Setup(s => s.GenerateConvertedMaintenancePlantReport(It.IsAny<KpiFormulaTarget>(), It.IsAny<KpiDetail>()))
                   .ReturnsAsync(_fixture.CreateMany<ConvertedKpiItemPlantDetails>(3).ToList());

        // Act
        var result = await serviceMock.Object.PerformanceSummaryPlantAsync(request, _fixture.Create<string>());

        // Assert
        Assert.NotNull(result);
        Assert.IsType<KpiPerformanceResponse>(result);
        Assert.True(result.average >= 0);
        Assert.True(result.bestPlant >= 0);
        Assert.NotNull(result.performanceSummary);
        Assert.True(result.target >= 0);
    }

    // ✅ 2. Empty costEffectivenessPlantList — test "if (Count > 0)" false branch
    [Fact]
    public async Task PerformanceSummaryPlantAsync_ShouldHandleEmptyCostEffectivenessList()
    {
        // Arrange
        var request = _fixture.Build<GetKpiPerformanceRequest>()
                              .With(x => x.affiliateId, _fixture.Create<int>())
                              .With(x => x.performanceSummary, _fixture.Create<string>())
                              .With(x => x.plantId, _fixture.Create<int>())
                              .Create();

        _currRepositoryMock.Setup(r => r.GetAffiliateLists()).ReturnsAsync(new List<AffiliateList>());
        _configRepositoryMock.Setup(r => r.GetCaseHierarchyAsync(It.IsAny<GetCaseHierarchyRequest>()))
                             .ReturnsAsync(new List<GetCaseHierarchyResponse>());

        var serviceMock = new Mock<PerformanceSummaryServices>(
            _currRepositoryMock.Object,
            _configRepositoryMock.Object,
            _performanceSumRepositoryMock.Object,
            new Mock<IConfiguration>().Object
        ) { CallBase = true };

        // Force empty allConvertedplantReports & allConvertedReport
        serviceMock.Setup(s => s.GetKpiFormulas(It.IsAny<KpiFormulaTargetRequest>()))
                   .Returns(new List<KpiFormulaTarget>());
        serviceMock.Setup(s => s.GetPerformanceSummaryActualDataNew(It.IsAny<GetKpiPerformanceRequest>(), It.IsAny<string>()))
                   .Returns(new Dictionary<string, List<KpiDetail>>());
        serviceMock.Setup(s => s.GetPerformanceSummaryAffiliateActualData(It.IsAny<GetKpiPerformanceRequest>(), It.IsAny<string>(), It.IsAny<List<GetCaseHierarchyResponse>>()))
                   .Returns(new Dictionary<string, List<KpiDetail>>());

        // Act
        var result = await serviceMock.Object.PerformanceSummaryPlantAsync(request, _fixture.Create<string>());

        // Assert
        Assert.NotNull(result);
        Assert.Null(result.bestPlantName);
        Assert.Equal(0, result.bestPlant);
        Assert.Equal(0, result.average);
        Assert.Equal(0, result.target);
    }

    // ✅ 3. Exception Handling Path — covers catch block
    [Fact]
    public async Task PerformanceSummaryPlantAsync_ShouldReturnDefaultResponse_WhenExceptionThrown()
    {
        // Arrange
        var request = _fixture.Create<GetKpiPerformanceRequest>();
        var serviceMock = new Mock<PerformanceSummaryServices>(
            _currRepositoryMock.Object,
            _configRepositoryMock.Object,
            _performanceSumRepositoryMock.Object,
            new Mock<IConfiguration>().Object
        ) { CallBase = true };

        // Simulate repository throwing exception
        _currRepositoryMock.Setup(r => r.GetAffiliateLists()).ThrowsAsync(new Exception("DB Failure"));

        // Act
        var result = await serviceMock.Object.PerformanceSummaryPlantAsync(request, _fixture.Create<string>());

        // Assert
        Assert.NotNull(result);
        Assert.IsType<KpiPerformanceResponse>(result);
        Assert.Null(result.kpis); // Default empty
    }
}
