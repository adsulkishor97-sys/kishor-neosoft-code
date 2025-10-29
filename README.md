[TestMethod]
public async Task GetMonthlyAndQuarterlyAsync_PIPoint_NotNull_FindPIPoint()
{
    // Arrange
    var fixture = new Fixture();

    var request = fixture.Build<AssetsPerformanceKpiTrendsRequest>()
        .With(r => r.sapId, fixture.Create<int>())
        .With(r => r.kpiCode, fixture.Create<int>())
        .With(r => r.startTime, DateTime.Now.AddMonths(-3))
        .With(r => r.endTime, DateTime.Now)
        .Create();

    var piTag = fixture.Create<string>();

    var fakeServer = new List<PIServer>();
    var server = (PIServer)FormatterServices.GetSafeUninitializedObject(typeof(PIServer));
    fakeServer.Add(server);

    _serverHelperMoke.Setup(x => x.ConnectedPIServers).Returns(fakeServer);

    // Mock a valid PIPoint
    var mockPiPoint = new Mock<PIPoint>();

    var summaryResult = new Dictionary<AFSummaryTypes, AFValue>
    {
        { AFSummaryTypes.Average, new AFValue(fixture.Create<double>()) }
    };

    // When SummaryAsync is called, return fake average value
    mockPiPoint
        .Setup(p => p.SummaryAsync(
            It.IsAny<AFTimeRange>(),
            AFSummaryTypes.Average,
            AFCalculationBasis.TimeWeighted,
            AFTimestampCalculation.Auto))
        .ReturnsAsync(summaryResult);

    var fakePiPoint = mockPiPoint.Object;

    // Setup FindPIPoint to return the mocked PIPoint
    _pIPointWrapper.Setup(x => x.FindPIPoint(server, piTag)).Returns(fakePiPoint);

    // Create service instance (same pattern as reference)
    var assetsService = new AssetsService(
        _assetsRepositoryMock.Object,
        _serverHelperMoke.Object,
        _pIPointWrapper.Object);

    // Act — invoke private method via reflection
    var methodInfo = typeof(AssetsService).GetMethod("GetSummariesAsync", BindingFlags.NonPublic | BindingFlags.Instance);
    Assert.IsNotNull(methodInfo, "Private method GetSummariesAsync not found.");

    var privateTask = (Task<IEnumerable<AssetsPerformanceKpiTrendsResponse>>)methodInfo.Invoke(
        assetsService,
        new object[] { fakePiPoint, request.startTime, request.endTime, "Month", 1 });

    var privateResult = await privateTask;

    // Assert private method behavior
    Assert.IsNotNull(privateResult);
    Assert.IsTrue(privateResult.Any());
    Assert.IsTrue(privateResult.All(r => r.frequency == "Month"));
    Assert.IsTrue(privateResult.All(r => !string.IsNullOrEmpty(r.tagName)));

    // Act — invoke public method
    var result = await assetsService.GetMonthlyAndQuarterlyAsync(piTag, request.startTime, request.endTime);

    // Assert public method behavior
    Assert.IsNotNull(result);
    Assert.IsTrue(result.Any(), "Expected non-empty list of KPI trend responses.");
    Assert.IsInstanceOfType(result, typeof(List<AssetsPerformanceKpiTrendsResponse>));
}
