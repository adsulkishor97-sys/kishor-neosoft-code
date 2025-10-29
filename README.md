[TestMethod]
public async Task GetMonthlyAndQuarterlyAsync_PIPoint_NotNull_FindPIPoint()
{
    // Arrange
    var fixture = new Fixture();

    var request = fixture.Build<AssetsPerformanceKpiTrendsRequest>()
        .With(r => r.startTime, DateTime.Now.AddMonths(-3))
        .With(r => r.endTime, DateTime.Now)
        .Create();

    var piTag = fixture.Create<string>();

    // Create fake PI Server instance
    var fakeServer = (PIServer)FormatterServices.GetSafeUninitializedObject(typeof(PIServer));
    var fakeServers = new List<PIServer> { fakeServer };

    _serverHelperMoke.Setup(x => x.ConnectedPIServers).Returns(fakeServers);

    // Create fake PIPoint instance (cannot mock sealed class)
    var fakePiPoint = (PIPoint)FormatterServices.GetSafeUninitializedObject(typeof(PIPoint));

    // Mock wrapper to return fake PIPoint
    _pIPointWrapper.Setup(x => x.FindPIPoint(fakeServer, piTag))
        .Returns(fakePiPoint);

    // Create fake AFValue result dictionary to simulate SummaryAsync() return
    var fakeSummaryDict = new Dictionary<AFSummaryTypes, AFValue>
    {
        { AFSummaryTypes.Average, new AFValue(12.34, new AFTime(DateTime.Now)) }
    };

    // Inject this fake result into GetSummariesAsync using reflection
    var assetService = new AssetsService(
        _assetsRepositoryMock.Object,
        _serverHelperMoke.Object,
        _pIPointWrapper.Object);

    // Use reflection to call private GetSummariesAsync()
    var privateMethod = typeof(AssetsService)
        .GetMethod("GetSummariesAsync", BindingFlags.NonPublic | BindingFlags.Instance);

    Assert.NotNull(privateMethod);

    // Invoke private method
    var privateTask = (Task)privateMethod.Invoke(
        assetService,
        new object[] { fakePiPoint, request.startTime, request.endTime, "Month", 1 });

    await privateTask.ConfigureAwait(false);

    var resultProp = privateTask.GetType().GetProperty("Result");
    var privateResult = resultProp?.GetValue(privateTask);

    // Assert private result
    Assert.NotNull(privateResult);
    Assert.IsInstanceOfType(privateResult, typeof(IEnumerable<AssetsPerformanceKpiTrendsResponse>));

    // Act: Call main public method
    var result = await assetService.GetMonthlyAndQuarterlyAsync(piTag, request.startTime, request.endTime);

    // Assert public result
    Assert.IsNotNull(result);
    Assert.IsInstanceOfType(result, typeof(List<AssetsPerformanceKpiTrendsResponse>));
}
