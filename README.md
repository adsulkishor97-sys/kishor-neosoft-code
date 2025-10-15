using System;
using System.Collections.Generic;
using System.Linq;
using System.Reflection;
using System.Threading.Tasks;
using AutoFixture;
using Moq;
using Xunit;

// Assuming these are your namespaces
using YourNamespace.Services;
using YourNamespace.Repository;
using YourNamespace.Models;

public class PerformanceSummaryAsyncTests
{
    private readonly IFixture _fixture;
    private readonly Mock<ICurrentRepository> _currRepositoryMock;
    private readonly Mock<IPerformanceSummaryRepository> _performanceSumRepositoryMock;
    private readonly PerformanceSummaryServices _performanceSummServices;

    public PerformanceSummaryAsyncTests()
    {
        _fixture = new Fixture();

        _currRepositoryMock = new Mock<ICurrentRepository>();
        _performanceSumRepositoryMock = new Mock<IPerformanceSummaryRepository>();

        _performanceSummServices = new PerformanceSummaryServices(
            _currRepositoryMock.Object,
            _performanceSumRepositoryMock.Object
        );
    }

    [Theory]
    [ClassData(typeof(PerformanceSummaryRequestGenerator))]
    public async Task PerformanceSummaryAsync_ShouldReturnData_WhenRepositoryReturnsData(
        GetKpiPerformanceRequest request,
        GetCaseHierarchyRequest affiliateIdrequest,
        List<AffiliateList> affiliateList,
        List<KpiFormulaTarget> dbFormulas)
    {
        // Arrange
        _currRepositoryMock.Setup(repo => repo.GetAffiliateLists())
                           .ReturnsAsync(affiliateList);

        _performanceSumRepositoryMock.Setup(repo => repo.GetKpiFormulaTarget(It.IsAny<KpiFormulaTargetRequest>()))
                                     .ReturnsAsync(dbFormulas);

        // Act
        var result = await _performanceSummServices.PerformanceSummaryAsync(request, affiliateIdrequest.affiliateIdList!);

        // Assert
        Assert.NotNull(result);
        Assert.IsType<KpiPerformanceResponse>(result);
        Assert.True(result.average >= 0);
    }

    [Fact]
    public async Task GenerateMaintenanceKpiReport_ShouldReturnValidKpiDetail_WhenValidDataProvided()
    {
        // Arrange
        var kpiName = _fixture.Create<string>();

        var affiliates = _fixture.Build<AffiliateList>()
                                 .With(x => x.affiliateCode, _fixture.Create<int>())
                                 .With(x => x.affiliateName, _fixture.Create<string>())
                                 .CreateMany(3)
                                 .ToList();

        var actuals = affiliates.ToDictionary(
            a => a.affiliateCode,
            a => _fixture.Create<decimal>() > 0 ? _fixture.Create<decimal>() : 1m // ensure >0
        );

        var targetValue = _fixture.Create<decimal>() > 0 ? _fixture.Create<decimal>() : 1m;

        // Use Reflection to access the private static method
        var method = typeof(PerformanceSummaryServices)
            .GetMethod("GenerateMaintenanceKpiReport", BindingFlags.NonPublic | BindingFlags.Static);

        Assert.NotNull(method);

        // Act
        var task = (Task<KpiDetail>)method!.Invoke(null, new object[] { kpiName, affiliates, actuals, targetValue })!;
        var result = await task;

        // Assert
        Assert.NotNull(result);
        Assert.Equal(kpiName, result.kpi);
        Assert.NotEmpty(result.affiliates);
        Assert.True(result.average >= 0);
        Assert.True(result.bestAffiliate >= 0);
        Assert.False(string.IsNullOrWhiteSpace(result.bestAffiliateName));
    }
}
