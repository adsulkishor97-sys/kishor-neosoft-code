[TestMethod]
public async Task GetMonthlyAndQuarterlyAsync_PIPoint_Catch_ThrowException_ReturnResponse()
{
    var fixture = new Fixture();
    var sapId = fixture.Create<int>();
    var kpiCode = fixture.Create<int>();
    var request = fixture.Build<AssetsPerformanceKpiTrendsRequest>()
        .With(r => r.sapId, sapId)
        .With(r => r.kpiCode, kpiCode)
        .With(r => r.startTime, DateTime.Now.AddMonths(-2))
        .With(r => r.endTime, DateTime.Now)
        .Create();

    var fackserver = new List<PIServer>();
    var pitag = fixture.Create<string>();
    var name = fixture.Create<string>();
    PIServer server = null;
   
    _pIServerWrapper.Setup(x => x.FindPIServer(name)).Returns(server);

    fackserver.Add(server);
    _serverHelperMoke.Setup(x => x.ConnectedPIServers).Returns(fackserver);
    _pIPointWrapper.Setup(x => x.FindPIPoint(server, pitag)).Throws(new Exception("failuer"));
    //Act
    var result = await _assetsService.GetMonthlyAndQuarterlyAsync(pitag, request.startTime, request.endTime);

    //Assert
    Assert.AreEqual(0, result.Count);
    Assert.IsInstanceOfType(result, typeof(List<AssetsPerformanceKpiTrendsResponse>));

}
