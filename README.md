using System;
using System.Collections.Generic;
using System.Linq;
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

        // Setup repository mocks
        _currRepositoryMock.Setup(x => x.GetAffiliateLists()).ReturnsAsync(affiliateList);
        _configRepositoryMock.Setup(x => x.GetCaseHierarchyAsync(It.IsAny<GetCaseHierarchyRequest>()))
                             .ReturnsAsync(getCaseHierarchyResponses);
        _performanceSummaryRepositoryMock.Setup(x => x.GetKpiFormulaTarget(It.IsAny<KpiFormulaTargetRequest>()))
                                         .ReturnsAsync(kpiFormulaTarget);

        // Mock internal methods by using a partial mock or internal helper logic if exposed
        // Here we rely on the private method being called indirectly through the flow

        // Act
        var result = await _service.PerformanceSummaryPlantAsync(request, _fixture.Create<string>());

        // Assert
        Assert.NotNull(result);
        Assert.IsType<KpiPerformanceResponse>(result);

        // Validate response structure
        Assert.True(result.bestPlantName == null || result.bestPlantName is string);
        Assert.True(result.bestPlant >= 0);
        Assert.True(result.average >= 0);
        Assert.True(result.target >= 0);
    }
}
