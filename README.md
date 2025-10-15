[Fact]
public async Task GenerateMaintenanceKpiReport_ShouldReturnValidKpiDetail_WhenValidDataProvided()
{
    // Arrange
    var kpiName = _fixture.Create<string>();

    var affiliates = _fixture.Build<AffiliateList>()
                             .With(x => x.affiliateName, _fixture.Create<string>())
                             .CreateMany(3)
                             .Select((a, index) => { a.affiliateCode = index + 1; return a; }) // ensure unique keys
                             .ToList();

    var actuals = affiliates.ToDictionary(
        a => a.affiliateCode,
        a => Math.Abs(_fixture.Create<decimal>()) + 1m // ensure >0 and unique
    );

    var targetValue = Math.Abs(_fixture.Create<decimal>()) + 1m;

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
    Assert.NotNull(result.affiliates);
    Assert.NotEmpty(result.affiliates!);
    Assert.True(result.average >= 0);
    Assert.True(result.bestAffiliate >= 0);
    Assert.False(string.IsNullOrWhiteSpace(result.bestAffiliateName));
}
