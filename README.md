[Fact]
public async Task GenerateConvertedMaintenanceReport_ShouldReturnConvertedList_WhenValidInputProvided()
{
    // Arrange
    var fixture = new Fixture();

    // Create mock affiliates inside KpiDetail
    var affiliates = fixture.Build<Affiliate>()
                            .With(a => a.affiliateName, fixture.Create<string>())
                            .With(a => a.actual, fixture.Create<decimal>() + 10m) // ensure > 0
                            .With(a => a.target, fixture.Create<decimal>())
                            .With(a => a.state, 1)
                            .CreateMany(3)
                            .ToList();

    var report = fixture.Build<KpiDetail>()
                        .With(r => r.affiliates, affiliates)
                        .Create();

    // Build KpiFormula with a simple valid lambda
    var kpiFormula = fixture.Build<KpiFormula>()
                            .With(f => f.formula, new Func<decimal, decimal>(x => x / 2m))
                            .Create();

    // Use Reflection to access private static method
    var method = typeof(PerformanceSummaryServices)
        .GetMethod("GenerateConvertedMaintenanceReport", BindingFlags.NonPublic | BindingFlags.Static);

    Assert.NotNull(method); // safety check

    // Act
    var task = (Task<List<ConvertedKpiItemDetails>>)method!.Invoke(null, new object[] { kpiFormula, report })!;
    var result = await task;

    // Assert
    Assert.NotNull(result);
    Assert.NotEmpty(result);
    Assert.All(result, item =>
    {
        Assert.False(string.IsNullOrWhiteSpace(item.affiliate));
        Assert.True(item.actualY >= 0);
        Assert.Equal("100%", item.target);
        Assert.Equal(0, item.min);
        Assert.Equal(2, item.max);
    });
}
