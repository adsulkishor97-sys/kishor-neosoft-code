[Fact]
public async Task GenerateMaintenancePlantKpiReport_ShouldReturnKpiDetail_WithOrderedPlants_WhenValidDataProvided()
{
    // Arrange
    var fixture = new Fixture();

    // Randomly create a KPI name
    var kpiName = fixture.Create<string>();

    // Generate random plant details
    var plantDetails = fixture.Build<GetCaseHierarchyResponse>()
                              .With(x => x.plantId, fixture.Create<int>())
                              .With(x => x.plantName, fixture.Create<string>())
                              .CreateMany(3)
                              .ToList();

    // Generate random actual values dictionary matching plant IDs
    var actuals = plantDetails.ToDictionary(
        p => (int)p.plantId!,
        p => fixture.Create<decimal>()
    );

    // Generate a random positive target value
    var targetValue = fixture.Create<decimal>() > 0 ? fixture.Create<decimal>() : 1m;

    // Use reflection to access the private static method
    var method = typeof(PerformanceSummaryServices)
        .GetMethod("GenerateMaintenancePlantKpiReport", BindingFlags.NonPublic | BindingFlags.Static);

    Assert.NotNull(method); // sanity check

    // Act: invoke method using reflection
    var task = (Task<KpiDetail>)method!.Invoke(null, new object[] { kpiName, plantDetails, actuals, targetValue })!;
    var result = await task;

    // Assert
    Assert.NotNull(result);
    Assert.Equal(kpiName, result.kpi);
    Assert.NotNull(result.plants);
    Assert.NotEmpty(result.plants);
    Assert.True(result.plants.All(p => p.actual >= 0));
    Assert.True(result.plants.All(p => p.target == targetValue));
    Assert.All(result.plants, p => Assert.False(string.IsNullOrWhiteSpace(p.plantName)));

    // Verify plants are ordered by actual ascending
    var ordered = result.plants.OrderBy(p => p.actual).ToList();
    Assert.Equal(ordered.Select(p => p.actual), result.plants.Select(p => p.actual));
}
