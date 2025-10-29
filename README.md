[Fact]
public async Task GetSummariesAsync_ShouldReturnExpectedResults_UsingReflection()
{
    // Arrange
    var fixture = new Fixture();

    // Create dependencies (mock PIPoint & others)
    var piPointMock = new Mock<PIPoint>();
    var afSummaryResult = new Dictionary<AFSummaryTypes, AFValue>
    {
        { AFSummaryTypes.Average, new AFValue(42.5) }
    };

    piPointMock.Setup(p => p.SummaryAsync(
        It.IsAny<AFTimeRange>(),
        AFSummaryTypes.Average,
        AFCalculationBasis.TimeWeighted,
        AFTimestampCalculation.Auto))
        .ReturnsAsync(afSummaryResult);

    var service = fixture.Create<AssetsPerformanceService>();

    // Set private dependencies via reflection if needed
    typeof(AssetsPerformanceService)
        .GetField("_pIPointWrapper", BindingFlags.NonPublic | BindingFlags.Instance)?
        .SetValue(service, new Mock<IPIPointWrapper>().Object);

    // Use reflection to get private method info
    var methodInfo = typeof(AssetsPerformanceService)
        .GetMethod("GetSummariesAsync", BindingFlags.NonPublic | BindingFlags.Instance);

    Assert.NotNull(methodInfo); // sanity check

    // Prepare parameters
    var piPoint = piPointMock.Object;
    var stime = DateTime.Now.AddMonths(-3);
    var etime = DateTime.Now;
    string period = "month";
    int stepMonths = 1;

    // Invoke private async method via reflection
    var task = (Task<IEnumerable<AssetsPerformanceKpiTrendsResponse>>)methodInfo.Invoke(
        service,
        new object[] { piPoint, stime, etime, period, stepMonths }
    );

    var result = await task;

    // Assert
    Assert.NotNull(result);
    Assert.NotEmpty(result);
    var first = result.First();
    Assert.Equal("month", first.frequency);
    Assert.Equal(piPointMock.Object.Name, first.tagName);
}
