[TestMethod]
public async Task GetMonthlyAndQuarterlyAsync_PIPoint_NotNull_FindPIPoint()
{
    // Arrange
    var fixture = new Fixture();

    // AutoFixture for request inputs
    var request = fixture.Build<AssetsPerformanceKpiTrendsRequest>()
        .With(r => r.startTime, DateTime.Now.AddMonths(-3))
        .With(r => r.endTime, DateTime.Now)
        .Create();

    var piTag = fixture.Create<string>();

    // Create fake PI Server
    var fakeServer = (PIServer)FormatterServices.GetSafeUninitializedObject(typeof(PIServer));
    var fakeServers = new List<PIServer> { fakeServer };

    _serverHelperMoke.Setup(x => x.ConnectedPIServers).Returns(fakeServers);

    // Fake PIPoint
    var fakePiPoint = new Mock<PIPoint>();
    fakePiPoint.Setup(x => x.Name).Returns(piTag);

    // Mock FindPIPoint to return fake point
    _pIPointWrapper.Setup(x => x.FindPIPoint(fakeServer, piTag))
        .Returns(fakePiPoint.Object);

    // Mock SummaryAsync indirectly using reflection safe override:
    var summaryDict = new Dictionary<AFSummaryTypes, AFValue>
    {
        {
            AFSummaryTypes.Average,
            new AFValue(12.34, new AFTime(DateTime.Now))
        }
    };

    // Instead of calling real SummaryAsync (which hits AF SDK), 
    // intercept via Setup on Mock<PIPoint>
    fakePiPoint
        .Setup(x => x.SummaryAsync(
            It.IsAny<AFTimeRange>(),
            It.IsAny<AFSummaryTypes>(),
            It.IsAny<AFCalculationBasis>(),
            It.IsAny<AFTimestampCalculation>(),
            It.IsAny<CancellationToken>()))
        .ReturnsAsync(summaryDict);

    // Service under test
    var assetService = new AssetsService(
        _assetsRepositoryMock.Object,
        _serverHelperMoke.Object,
        _pIPointWrapper.Object);

    // 👇 Access private method via reflection
    var privateMethod = typeof(AssetsService)
        .GetMethod("GetSummariesAsync", BindingFlags.NonPublic | BindingFlags.Instance);

    var privateTask = (Task)privateMethod.Invoke(
        assetService,
        new object[] { fakePiPoint.Object, request.startTime, request.endTime, "Month", 1 });

    await privateTask.ConfigureAwait(false);
    var resultProperty = privateTask.GetType().GetProperty("Result");
    var privateResult = resultProperty?.GetValue(privateTask);

    // Assert private result
    Assert.NotNull(privateResult);
    var list = privateResult as IEnumerable<AssetsPerformanceKpiTrendsResponse>;
    Assert.IsTrue(list!.Any());

    // Act - test the public method
    var result = await assetService.GetMonthlyAndQuarterlyAsync(piTag, request.startTime, request.endTime);

    // Assert public method output
    Assert.NotNull(result);
    Assert.IsInstanceOfType(result, typeof(List<AssetsPerformanceKpiTrendsResponse>));
    Assert.IsTrue(result.Count > 0);
}
