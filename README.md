using System;
using System.Collections.Generic;
using System.Linq;
using System.Reflection;
using System.Threading.Tasks;
using Xunit;
using AutoFixture;

public class GenerateConvertedMaintenancePlantReportTests
{
    private readonly IFixture _fixture;

    public GenerateConvertedMaintenancePlantReportTests()
    {
        _fixture = new Fixture();
    }

    [Fact]
    public async Task GenerateConvertedMaintenancePlantReport_ShouldReturnConvertedList_WhenPlantsExist()
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

        // Act
        var result = await InvokeGenerateConvertedMaintenancePlantReport(kpiFormula, report);

        // Assert
        Assert.NotNull(result);
        Assert.Equal(report.plants.Count, result.Count);
        Assert.All(result, item =>
        {
            Assert.NotNull(item.plant);
            Assert.True(item.actualY >= 0);
            Assert.Equal("100%", item.target);
            Assert.Equal(0, item.min);
            Assert.Equal(2, item.max);
        });
    }

    [Fact]
    public async Task GenerateConvertedMaintenancePlantReport_ShouldHandleNullPlantsList()
    {
        // Arrange
        var report = _fixture.Build<KpiDetail>()
                             .With(r => r.plants, (List<Plant>?)null)
                             .Create();

        var kpiFormula = _fixture.Build<KpiFormula>()
                                 .With(f => f.formula, new Func<decimal, decimal>(x => x * 1.5m))
                                 .Create();

        // Act
        var result = await InvokeGenerateConvertedMaintenancePlantReport(kpiFormula, report);

        // Assert
        Assert.NotNull(result);
        Assert.Empty(result); // Should return empty list safely
    }

    [Fact]
    public async Task GenerateConvertedMaintenancePlantReport_ShouldConvertNegativeValuesToZero()
    {
        // Arrange
        var plant = _fixture.Build<Plant>()
                            .With(p => p.actual, _fixture.Create<decimal>())
                            .With(p => p.plantName, _fixture.Create<string>())
                            .Create();

        var report = _fixture.Build<KpiDetail>()
                             .With(r => r.plants, new List<Plant> { plant })
                             .Create();

        var kpiFormula = _fixture.Build<KpiFormula>()
                                 .With(f => f.formula, new Func<decimal, decimal>(x => -Math.Abs(x))) // always negative
                                 .Create();

        // Act
        var result = await InvokeGenerateConvertedMaintenancePlantReport(kpiFormula, report);

        // Assert
        Assert.NotNull(result);
        Assert.Single(result);
        Assert.Equal(0, result.First().actualY); // negative converted to zero
    }

    // ✅ Helper method: invokes the private static method using reflection
    private static Task<List<ConvertedKpiItemPlantDetails>> InvokeGenerateConvertedMaintenancePlantReport(
        KpiFormula kpiFormula,
        KpiDetail report)
    {
        var method = typeof(YourClassNameHere) // 🔁 Replace with the class where method is defined
            .GetMethod("GenerateConvertedMaintenancePlantReport",
                BindingFlags.NonPublic | BindingFlags.Static);

        return (Task<List<ConvertedKpiItemPlantDetails>>)method!.Invoke(null, new object[] { kpiFormula, report })!;
    }
}
