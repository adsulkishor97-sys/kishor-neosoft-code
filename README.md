using Xunit;
using Moq;
using AutoFixture;
using System.Collections.Generic;
using System.Threading.Tasks;
using System.Linq;
using YourNamespace.Services;      // replace with actual namespace
using YourNamespace.Repositories;  // replace with actual namespace
using YourNamespace.Models;        // replace with actual models

public class PerformanceSummaryPlantAsyncTests
{
    private readonly Mock<IConfigRepository> _configRepositoryMock;
    private readonly Mock<ICurrentRepository> _currRepositoryMock;
    private readonly Mock<IPerformanceSummaryRepository> _performanceSumRepositoryMock;
    private readonly PerformanceSummaryService _performanceSummServices;
    private readonly IFixture _fixture;

    public PerformanceSummaryPlantAsyncTests()
    {
        _fixture = new Fixture();

        _configRepositoryMock = new Mock<IConfigRepository>();
        _currRepositoryMock = new Mock<ICurrentRepository>();
        _performanceSumRepositoryMock = new Mock<IPerformanceSummaryRepository>();

        _performanceSummServices = new PerformanceSummaryService(
            _configRepositoryMock.Object,
            _currRepositoryMock.Object,
            _performanceSumRepositoryMock.Object
        );
    }

    [Fact]
    public async Task PerformanceSummaryPlantAsync_ShouldReturn_ValidResponse_WhenDataExists()
    {
        // Arrange
        var request = _fixture.Build<GetKpiPerformanceRequest>()
                              .With(x => x.performanceSummary, "TestSummary")
                              .Create();

        var affiliateIdRequest = _fixture.Build<GetCaseHierarchyRequest>()
                                         .With(x => x.affiliateIdList, request.affiliateId.ToString())
                                         .Create();

        var affiliateList = _fixture.CreateMany<AffiliateList>(3).ToList();
        var dbFormulas = _fixture.CreateMany<KpiFormulaTarget>(3).ToList();
        var getCaseHierarchyResponses = _fixture.CreateMany<GetCaseHierarchyResponse>(2).ToList();

        _currRepositoryMock
            .Setup(r => r.GetAffiliateLists())
            .ReturnsAsync(affiliateList);

        _configRepositoryMock
            .Setup(r => r.GetCaseHierarchyAsync(It.IsAny<GetCaseHierarchyRequest>()))
            .ReturnsAsync(getCaseHierarchyResponses);

        _performanceSumRepositoryMock
            .Setup(r => r.GetKpiFormulaTarget(It.IsAny<KpiFormulaTargetRequest>()))
            .ReturnsAsync(dbFormulas);

        // Mock internal service methods if they are virtual or extracted to helpers
        var affiliateRequestString = _fixture.Create<string>();

        // Act
        var result = await _performanceSummServices.PerformanceSummaryPlantAsync(request, affiliateRequestString);

        // Assert
        Assert.NotNull(result);
        Assert.IsType<KpiPerformanceResponse>(result);
    }

    [Fact]
    public async Task PerformanceSummaryPlantAsync_ShouldReturn_EmptyResponse_OnException()
    {
        // Arrange
        var request = _fixture.Create<GetKpiPerformanceRequest>();
        var affiliateRequest = _fixture.Create<string>();

        // Force repository to throw
        _configRepositoryMock
            .Setup(r => r.GetCaseHierarchyAsync(It.IsAny<GetCaseHierarchyRequest>()))
            .ThrowsAsync(new System.Exception("DB Error"));

        // Act
        var result = await _performanceSummServices.PerformanceSummaryPlantAsync(request, affiliateRequest);

        // Assert
        Assert.NotNull(result);
        Assert.IsType<KpiPerformanceResponse>(result);
        Assert.Null(result.kpis); // empty object on exception
    }
}
