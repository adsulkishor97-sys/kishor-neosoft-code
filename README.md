using Xunit;
using Moq;
using Moq.Protected;
using AutoFixture;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Reflection;
using System.Threading.Tasks;
using YourNamespace.Services; // replace with your actual namespace
using YourNamespace.Models;   // replace with actual model namespace

public class PerformanceSummaryServicesTests
{
    private readonly Fixture _fixture;

    public PerformanceSummaryServicesTests()
    {
        _fixture = new Fixture();
    }

    [Fact]
    public async Task GetPerformanceSummaryActualDataNew_ShouldReturnData_ForAllPerformanceSummaries()
    {
        // Arrange
        var serviceMock = new Mock<PerformanceSummaryServices> { CallBase = true };

        // Mock internal method GetPerformanceSummaryNumDenCalc
        var fakeCalcResult = _fixture.CreateMany<KpiNumeratorDenominatorPerformanceSummary>(3).ToList();

        serviceMock.Protected()
            .Setup<Task<List<KpiNumeratorDenominatorPerformanceSummary>>>(
                "GetPerformanceSummaryNumDenCalc",
                ItExpr.IsAny<string>(),
                ItExpr.IsAny<string>())
            .ReturnsAsync(fakeCalcResult);

        // Prepare DateTimeFormatRequest dynamically
        var dateTimeFormatRequest = _fixture.Build<DateTimeFormatRequest>()
            .With(x => x.startDateyyyymmdd, _fixture.Create<string>())
            .With(x => x.endDateyyyymmdd, _fixture.Create<string>())
            .With(x => x.startDateyyyymm, _fixture.Create<string>())
            .With(x => x.endDateyyyymm, _fixture.Create<string>())
            .With(x => x.affiliateRequest, _fixture.Create<string>())
            .Create();

        // Prepare fake GlobalPerformanceSummary data
        var globalData = _fixture.Build<GlobalPerformanceSummary>()
            .With(x => x.maintananceExcellence, _fixture.CreateMany<MaintananceExcellence>(2).ToList())
            .With(x => x.costEffectiveness, _fixture.CreateMany<MaintananceExcellence>(2).ToList())
            .With(x => x.assetPerformance, _fixture.CreateMany<MaintananceExcellence>(2).ToList())
            .Create();

        // Ensure Numerator and Denominator strings contain placeholders
        foreach (var list in new[] { globalData.maintananceExcellence, globalData.costEffectiveness, globalData.assetPerformance })
        {
            foreach (var item in list)
            {
                item.Numerator = "SELECT * FROM Table WHERE StartDateyyyymmdd='x' AND EndDateyyyymmdd='y'";
                item.Denominator = "SELECT * FROM Table WHERE StartDateyyyymm='x' AND EndDateyyyymm='y'";
            }
        }

        // Inject GlobalPerformanceSummary via reflection if field exists
        var globalField = typeof(PerformanceSummaryServices)
            .GetFields(BindingFlags.NonPublic | BindingFlags.Instance | BindingFlags.Static)
            .FirstOrDefault(f => f.FieldType == typeof(GlobalPerformanceSummary));

        if (globalField != null)
            globalField.SetValue(serviceMock.Object, globalData);

        // Dynamically create performance summary constants
        var kpiConstants = new Dictionary<string, string>
        {
            { nameof(KpiConstant.MaintananceExcellence), _fixture.Create<string>() },
            { nameof(KpiConstant.CostEffectivness), _fixture.Create<string>() },
            { nameof(KpiConstant.AssetPerformance), _fixture.Create<string>() }
        };

        // Inject AutoFixture values into static KpiConstant fields
        foreach (var kvp in kpiConstants)
        {
            typeof(KpiConstant)
                .GetField(kvp.Key, BindingFlags.Public | BindingFlags.Static)
                ?.SetValue(null, kvp.Value);
        }

        // Use reflection to call private method
        var method = typeof(PerformanceSummaryServices).GetMethod(
            "GetPerformanceSummaryActualDataNew",
            BindingFlags.NonPublic | BindingFlags.Instance);

        Assert.NotNull(method); // Sanity check

        // Act & Assert for all branches
        foreach (var kvp in kpiConstants)
        {
            var request = _fixture.Build<GetKpiPerformanceRequest>()
                .With(x => x.performanceSummary, kvp.Value)
                .Create();

            var task = (Task<List<KpiNumeratorDenominatorPerformanceSummary>>)method.Invoke(
                serviceMock.Object,
                new object[] { request, dateTimeFormatRequest })!;

            var result = await task;

            // Assert
            Assert.NotNull(result);
            Assert.NotEmpty(result);
            Assert.All(result, r => Assert.NotNull(r));

            // Verify internal async method was called
            serviceMock.Protected().Verify(
                "GetPerformanceSummaryNumDenCalc",
                Times.AtLeastOnce(),
                ItExpr.IsAny<string>(),
                ItExpr.IsAny<string>());
        }
    }
}
