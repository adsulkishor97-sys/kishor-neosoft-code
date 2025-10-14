using System;
using System.Collections.Generic;
using System.Reflection;
using System.Threading.Tasks;
using AutoFixture;
using Xunit;

public class KpiReportTests
{
    private readonly Fixture _fixture = new Fixture();

    [Fact]
    public async Task GenerateConvertedMaintenancePlantReport_ShouldReturn_ConvertedList()
    {
        // Arrange
        var kpiFormula = _fixture.Build<KpiFormula>()
                                 .With(k => k.formula, new Func<decimal, decimal>(x => x / 100m)) // positive values
                                 .Create();

        var report = _fixture.Build<KpiDetail>()
                             .With(r => r.plants, _fixture.CreateMany<Plant>(3).ToList())
                             .Create();

        // Act
        var method = typeof(YourClassContainingMethod)
                     .GetMethod("GenerateConvertedMaintenancePlantReport", BindingFlags.NonPublic | BindingFlags.Static);

        var task = (Task<List<ConvertedKpiItemPlantDetails>>)method!.Invoke(null, new object[] { kpiFormula, report })!;
        var result = await task;

        // Assert
        Assert.NotNull(result);
        Assert.Equal(report.plants.Count, result.Count);

        foreach (var converted in result)
        {
            Assert.NotNull(converted.plant);
            Assert.InRange(converted.actualY, 0, decimal.MaxValue);
            Assert.Equal("100%", converted.target);
            Assert.Equal(0, converted.min);
            Assert.Equal(2, converted.max);
        }
    }

    [Fact]
    public async Task GenerateConvertedMaintenancePlantReport_ShouldSet_NegativeYValueToZero()
    {
        // Arrange
        var kpiFormula = _fixture.Build<KpiFormula>()
                                 .With(k => k.formula, new Func<decimal, decimal>(x => x - 100)) // produce negative
                                 .Create();

        var report = _fixture.Build<KpiDetail>()
                             .With(r => r.plants, new List<Plant>
                             {
                                 _fixture.Build<Plant>().With(p => p.actual, 50).Create()
                             })
                             .Create();

        // Act
        var method = typeof(YourClassContainingMethod)
                     .GetMethod("GenerateConvertedMaintenancePlantReport", BindingFlags.NonPublic | BindingFlags.Static);

        var task = (Task<List<ConvertedKpiItemPlantDetails>>)method!.Invoke(null, new object[] { kpiFormula, report })!;
        var result = await task;

        // Assert
        Assert.Single(result);
        Assert.Equal(0, result[0].actualY); // negative branch covered
    }

    [Fact]
    public async Task GenerateConvertedMaintenancePlantReport_ShouldHandle_NullPlantsList()
    {
        // Arrange
        var kpiFormula = _fixture.Build<KpiFormula>()
                                 .With(k => k.formula, new Func<decimal, decimal>(x => x / 100m))
                                 .Create();

        var report = _fixture.Build<KpiDetail>()
                             .With(r => r.plants, (List<Plant>?)null)
                             .Create();

        // Act
        var method = typeof(YourClassContainingMethod)
                     .GetMethod("GenerateConvertedMaintenancePlantReport", BindingFlags.NonPublic | BindingFlags.Static);

        var task = (Task<List<ConvertedKpiItemPlantDetails>>)method!.Invoke(null, new object[] { kpiFormula, report })!;
        var result = await task;

        // Assert
        Assert.NotNull(result);
        Assert.Empty(result); // Null plants handled
    }
}
