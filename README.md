[TestMethod]
public async Task GetMonthlyAndQuarterlyAsync_PIPoint_NotNull_FindPIPoint()
{
    // Arrange
    var fixture = new Fixture();

    var request = fixture.Build<AssetsPerformanceKpiTrendsRequest>()
        .With(r => r.sapId, fixture.Create<int>())
        .With(r => r.kpiCode, fixture.Create<int>())
        .With(r => r.startTime, DateTime.Now.AddMonths(-2))
        .With(r => r.endTime, DateTime.Now)
        .Create();

    var fakeServer = new List<PIServer>();
    var server = (PIServer)FormatterServices.GetSafeUninitializedObject(typeof(PIServer));
    fakeServer.Add(server);
    _serverHelperMoke.Setup(x => x.ConnectedPIServers).Returns(fakeServer);

    var fakePiPointMock = new Mock<PIPoint>();
    var summaryResult = new Dictionary<AFSummaryTypes, AFValue>
    {
        { AFSummaryTypes.Average, new AFValue(25.5) }
    };

    fakePiPointMock.Setup(p => p.SummaryAsync(
        It.IsAny<AFTimeRange>(),
        AFSummaryTypes.Average,
        AFCalculationBasis.TimeWeighted,
        AFTimestampCalculation.Auto))
        .ReturnsAsync(summaryResult);

    var fakePiPoint = fakePiPointMock.Object;
    var piTag = "AR.AR1.P-852A.Equipment_Offline_Status_AMH_CALC_OUTPUT";

    _pIPointWrapper.Setup(x => x.FindPIPoint(server, piTag)).Returns(fakePiPoint);

    var assetsService = new AssetsService(
        _assetsRepositoryMock.Object,
        _serverHelperMoke.Object,
        _pIPointWrapper.Object);

    // Act: Invoke private method via reflection
    var methodInfo = typeof(AssetsService).GetMethod("GetSummariesAsync", BindingFlags.NonPublic | BindingFlags.Instance);
    var task = (Task<IEnumerable<AssetsPerformanceKpiTrendsResponse>>)methodInfo.Invoke(
        assetsService,
        new object[] { fakePiPoint, request.startTime, request.endTime, "Month", 1 });

    var privateResult = await task;

    // Assert private method output
    Assert.IsNotNull(privateResult);
    Assert.IsTrue(privateResult.Any());
    Assert.AreEqual("Month", privateResult.First().frequency);

    // Act: test public method as well
    var result = await assetsService.GetMonthlyAndQuarterlyAsync(piTag, request.startTime, request.endTime);

    // Assert
    Assert.IsNotNull(result);
    Assert.IsTrue(result.Any());
}
