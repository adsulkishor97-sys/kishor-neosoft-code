[Fact]
public async Task GenerateMaintenancePlantKpiReport_ShouldReturnKpiDetail_WithOrderedPlants_WhenValidDataProvided()
{
    // Arrange
    var fixture = new Fixture();

    // Random KPI name
    var kpiName = fixture.Create<string>();

    // Generate unique plant details (avoid duplicate plantId)
    var plantDetails = fixture.Build<GetCaseHierarchyResponse>()
                              .Without(x => x.plantId)
                              .With(x => x.plantName, () => fixture.Create<string>())
                              .CreateMany(3)
                              .Select((p, index) => 
                              {
                                  p.plantId = index + 1; // assign unique IDs
                                  return p;
                              })
                              .ToList();

    // Now all plantIds are unique, so dictionary won't throw
    var actuals = plantDetails.ToDictionary(
        p => (int)p.plantId!,
        p => fixture.Create<decimal>()
    );

    var targetValue = Math.Abs(fixture.Create<decimal>()); // ensure positive

    // Use reflection to access the private static method
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

    // Verify plants are ordered ascending by actual
    var ordered = result.plants.OrderBy(p => p.actual).ToList();
    Assert.Equal(ordered.Select(p => p.actual), result.plants.Select(p => p.actual));
}
