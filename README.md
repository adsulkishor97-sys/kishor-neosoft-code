using System;
using System.Collections.Generic;
using System.Reflection;
using System.Threading.Tasks;
using AutoFixture;
using Moq;
using Xunit;

public class HandleAffiliateDistributionNonGlobalPageRequestTests
{
    private readonly IFixture _fixture = new Fixture();
    private readonly Mock<ICurrentRepository> _currentRepositoryMock = new Mock<ICurrentRepository>();
    private readonly object _serviceInstance;

    public HandleAffiliateDistributionNonGlobalPageRequestTests()
    {
        // Create instance of your target service class using reflection
        // Update type name if needed
        var targetType = typeof(PerformanceSummaryServices);
        _serviceInstance = Activator.CreateInstance(targetType, _currentRepositoryMock.Object)!;
    }

    [Fact]
    public async Task HandleAffiliateDistributionNonGlobalPageRequest_ShouldExecuteSuccessfully_UsingAutoFixture()
    {
        // Arrange
        var request = _fixture.Build<GetAffiliatePlantDistributionRequest>()
                              .With(x => x.kpiCode, _fixture.Create<string>())
                              .With(x => x.affiliateId, _fixture.Create<List<int>>())
                              .Create();

        var affiliateRequest = _fixture.Create<string>();
        var formattedStartDate = _fixture.Create<DateTime>().ToString("yyyy-MM-dd");
        var formattedEndDate = _fixture.Create<DateTime>().AddDays(5).ToString("yyyy-MM-dd");
        var quotedPmCodes = $"'{_fixture.Create<string>()}','{_fixture.Create<string>()}'";

        var fakeHierarchy = _fixture.CreateMany<GetCaseHierarchyResponse>(3).ToList();
        var fakeNumData = _fixture.CreateMany<KpiNumeratorDenominatorAffiliateDistribution>(3).ToList();
        var fakeDenData = _fixture.CreateMany<KpiNumeratorDenominatorAffiliateDistribution>(3).ToList();
        var fakeGroupedData = _fixture.CreateMany<GroupedData>(2).ToList();

        // Mock repository behavior (no hardcoded values)
        _currentRepositoryMock.Setup(x => x.GetCaseHierarchyAsync(It.IsAny<List<int>>()))
                              .ReturnsAsync(fakeHierarchy);

        _currentRepositoryMock.Setup(x => x.ExecuteBigDataQuery_New<KpiNumeratorDenominatorAffiliateDistribution>(
            It.IsAny<string>(),
            It.IsAny<Func<System.Data.IDataReader, KpiNumeratorDenominatorAffiliateDistribution>>()))
            .ReturnsAsync(fakeNumData);

        // Access private method using reflection
        var targetType = _serviceInstance.GetType();
        var methodInfo = targetType.GetMethod("HandleAffiliateDistributionNonGlobalPageRequest",
            BindingFlags.NonPublic | BindingFlags.Instance);

        Assert.NotNull(methodInfo); // ensure reflection worked

        // Act
        var task = (Task<List<GroupedData>>)methodInfo!.Invoke(_serviceInstance, new object[]
        {
            request,
            affiliateRequest,
            formattedStartDate,
            formattedEndDate,
            quotedPmCodes
        })!;

        var result = await task;

        // Assert
        Assert.NotNull(result);
        Assert.IsType<List<GroupedData>>(result);
    }
}

[Fact]
public async Task GenerateMaintenancePlantKpiReport_ShouldReturnKpiDetail_WithOrderedPlants_WhenValidDataProvided()
{
    // Arrange
    var fixture = new Fixture();
    var kpiName = fixture.Create<string>();
    var plantDetails = fixture.Build<GetCaseHierarchyResponse>()
                              .Without(x => x.plantId)
                              .With(x => x.plantName, () => fixture.Create<string>())
                              .CreateMany(3)
                              .Select((p, index) =>
                              {
                                  p.plantId = index + 1; 
                                  return p;
                              })
                              .ToList();
    var actuals = plantDetails.ToDictionary(
        p => (int)p.plantId!,
        p => fixture.Create<decimal>()
    );
    var targetValue = Math.Abs(fixture.Create<decimal>()); 
    var method = typeof(PerformanceSummaryServices)
        .GetMethod("GenerateMaintenancePlantKpiReport", BindingFlags.NonPublic | BindingFlags.Static);
    Assert.NotNull(method);

    // Act
    var task = (Task<KpiDetail>)method!.Invoke(null, new object[] { kpiName, plantDetails, actuals, targetValue })!;
    var result = await task;

    // Assert
    Assert.NotNull(result);
    Assert.Equal(kpiName, result.kpi);
    Assert.NotNull(result.plants);
    Assert.NotEmpty(result.plants);
    Assert.All(result.plants, p => Assert.False(string.IsNullOrWhiteSpace(p.plantName)));
    Assert.All(result.plants, p => Assert.True(p.actual >= 0));
    Assert.All(result.plants, p => Assert.Equal(targetValue, p.target));
    var ordered = result.plants.OrderBy(p => p.actual).ToList();
    Assert.Equal(ordered.Select(p => p.actual), result.plants.Select(p => p.actual));
}
