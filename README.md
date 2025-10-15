using System;
using System.Collections.Generic;
using System.Linq;
using System.Reflection;
using System.Threading.Tasks;
using AutoFixture;
using Moq;
using Xunit;

// Assuming these are your namespaces:
using AMHDomain.Models.Current;
using AMHDomain.Models.Central;
using YourNamespace.Services;
using YourNamespace.Repository;

public class PerformanceSummaryPlantAsyncTests
{
    private readonly IFixture _fixture;
    private readonly Mock<ICurrentRepository> _currRepositoryMock;
    private readonly Mock<IConfigRepository> _configRepositoryMock;
    private readonly Mock<IPerformanceSummaryRepository> _performanceSummaryRepositoryMock;
    private readonly PerformanceSummaryServices _service;

    public PerformanceSummaryPlantAsyncTests()
    {
        _fixture = new Fixture();

        _currRepositoryMock = new Mock<ICurrentRepository>();
        _configRepositoryMock = new Mock<IConfigRepository>();
        _performanceSummaryRepositoryMock = new Mock<IPerformanceSummaryRepository>();

        _service = new PerformanceSummaryServices(
            _currRepositoryMock.Object,
            _configRepositoryMock.Object,
            _performanceSummaryRepositoryMock.Object
        );
    }

    [Fact]
    public async Task PerformanceSummaryPlantAsync_ShouldReturnValidResponse_WhenDependenciesReturnData()
    {
        // Arrange
        var request = _fixture.Build<GetKpiPerformanceRequest>()
                              .With(x => x.affiliateId, _fixture.Create<int>())
                              .With(x => x.performanceSummary, _fixture.Create<string>())
                              .With(x => x.plantId, _fixture.Create<int>())
                              .Create();

        var affiliateList = _fixture.CreateMany<AffiliateList>(3).ToList();
        var getCaseHierarchyResponses = _fixture.CreateMany<GetCaseHierarchyResponse>(3).ToList();
        var kpiFormulaTarget = _fixture.Build<KpiFormulaTarget>()
                                       .With(x => x.name, _fixture.Create<string>())
                                       .With(x => x.target, _fixture.Create<string>())
                                       .CreateMany(2).ToList();

        // Mock repository responses
        _currRepositoryMock.Setup(x => x.GetAffiliateLists()).ReturnsAsync(affiliateList);
        _configRepositoryMock.Setup(x => x.GetCaseHierarchyAsync(It.IsAny<GetCaseHierarchyRequest>()))
                             .ReturnsAsync(getCaseHierarchyResponses);
        _performanceSummaryRepositoryMock.Setup(x => x.GetKpiFormulaTarget(It.IsAny<KpiFormulaTargetRequest>()))
                                         .ReturnsAsync(kpiFormulaTarget);

        // Act
        var result = await _service.PerformanceSummaryPlantAsync(request, _fixture.Create<string>());

        // Assert
        Assert.NotNull(result);
        Assert.IsType<KpiPerformanceResponse>(result);
        Assert.True(result.bestPlant >= 0);
    }

    [Fact]
    public async Task GenerateConvertedMaintenancePlantReport_ShouldReturnConvertedValues_WhenPlantsExist()
    {
        // Arrange
        var plants = _fixture.Build<Plant>()
                             .With(p => p.actual, _fixture.Create<decimal>())
                             .With(p => p.plantName, _fixture.Create<string>())
                             .CreateMany(3)
                             .ToList();

        var report = _fixture.Build<KpiDetail>()
                             .With(x => x.plants, plants)
                             .Create();

        var kpiFormula = _fixture.Build<KpiFormula>()
                                 .With(f => f.formula, new Func<decimal, decimal>(x => x + _fixture.Create<decimal>()))
                                 .Create();

        // Use reflection to call the private static method
        var method = typeof(PerformanceSummaryServices)
                     .GetMethod("GenerateConvertedMaintenancePlantReport", BindingFlags.NonPublic | BindingFlags.Static);

        Assert.NotNull(method);

        var task = (Task<List<ConvertedKpiItemPlantDetails>>)method!.Invoke(null, new object[] { kpiFormula, report })!;
        var result = await task;

        // Assert
        Assert.NotNull(result);
        Assert.Equal(report.plants.Count, result.Count);
        Assert.All(result, item =>
        {
            Assert.False(string.IsNullOrWhiteSpace(item.plant));
            Assert.True(item.actualY >= 0);
            Assert.Equal("100%", item.target);
            Assert.Equal(0, item.min);
            Assert.Equal(2, item.max);
        });
    }
}
