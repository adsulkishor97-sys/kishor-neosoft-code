[Fact]
public async Task GenerateConvertedMaintenancePlantReport_ShouldReturn_ConvertedList()
{
    // Arrange
    var fixture = new Fixture();

    // Create a fake KPI formula
    var kpiFormula = fixture.Build<KpiFormula>()
                            .With(k => k.formula, new Func<decimal, decimal>(x => x / 100m)) // simple conversion
                            .Create();

    // Create a fake KpiDetail with plants
    var report = fixture.Build<KpiDetail>()
                        .With(r => r.plants, fixture.CreateMany<Plant>(3).ToList())
                        .Create();

    // Act: use reflection to call private static method
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
