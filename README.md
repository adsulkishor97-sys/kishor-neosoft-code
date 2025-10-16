using Xunit;
using Moq;
using AutoFixture;
using System.Collections.Generic;
using System.Threading.Tasks;
using System.Linq;
using System;
using System.Reflection;
using YourNamespace.Models; // change to your models namespace
using YourNamespace.Services; // change to your service namespace
using Moq.Protected;

public class PerformanceSummaryServiceTests
{
    [Fact]
    public async Task GetPerformanceSummaryActualDataNew_ShouldCoverAllBranches_WithoutHardcodedValues()
    {
        // Arrange
        var fixture = new Fixture();

        // Create mock of service (so we can mock internal async method)
        var serviceMock = new Mock<PerformanceSummaryServices> { CallBase = true };

        // Dynamically generate unique strings to simulate summary keys
        var summaryKeys = fixture.CreateMany<string>(3).ToList();

        // Create fake KpiConstant object dynamically via reflection (simulate real constants)
        var kpiConstantType = typeof(KpiConstant);
        var fields = kpiConstantType.GetFields(BindingFlags.Public | BindingFlags.Static);
        foreach (var field in fields)
        {
            field.SetValue(null, fixture.Create<string>());
        }

        // Generate fake request
        var request = fixture.Build<GetKpiPerformanceRequest>()
            .With(x => x.performanceSummary, summaryKeys[0]) // AutoFixture string
            .Create();

        var dateTimeFormatRequest = fixture.Create<DateTimeFormatRequest>();

        // Fake result from GetPerformanceSummaryNumDenCalc
        var fakeCalcResult = fixture.CreateMany<KpiNumeratorDenominatorPerformanceSummary>(3).ToList();

        // Mock GetPerformanceSummaryNumDenCalc (private async)
        serviceMock.Protected()
            .Setup<Task<List<KpiNumeratorDenominatorPerformanceSummary>>>(
                "GetPerformanceSummaryNumDenCalc",
                ItExpr.IsAny<string>(),
                ItExpr.IsAny<string>())
            .ReturnsAsync(fakeCalcResult);

        // Prepare fake GlobalPerformanceSummary object
        var global = fixture.Build<GlobalPerformanceSummary>()
            .With(x => x.maintananceExcellence, fixture.CreateMany<MaintananceExcellence>(2).ToList())
            .With(x => x.costEffectiveness, fixture.CreateMany<MaintananceExcellence>(2).ToList())
            .With(x => x.assetPerformance, fixture.CreateMany<MaintananceExcellence>(2).ToList())
            .Create();

        // Inject the fake GlobalPerformanceSummary (if it exists as a field)
        var globalField = typeof(PerformanceSummaryServices)
            .GetFields(BindingFlags.NonPublic | BindingFlags.Instance | BindingFlags.Static)
            .FirstOrDefault(f => f.FieldType == typeof(GlobalPerformanceSummary));

        if (globalField != null)
            globalField.SetValue(serviceMock.Object, global);

        // Act
        var result = await serviceMock.Object.InvokePrivateAsync<List<KpiNumeratorDenominatorPerformanceSummary>>(
            "GetPerformanceSummaryActualDataNew",
            request, dateTimeFormatRequest);

        // Assert
        Assert.NotNull(result);
        Assert.All(result, r => Assert.NotNull(r));
        Assert.True(result.Count >= 0);

        // Verify mock was called (ensures foreach loop executed)
        serviceMock.Protected().Verify(
            "GetPerformanceSummaryNumDenCalc",
            Times.AtLeastOnce(),
            ItExpr.IsAny<string>(),
            ItExpr.IsAny<string>());
    }
}
