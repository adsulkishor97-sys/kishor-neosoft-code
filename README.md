 [TestMethod]
 public async Task GetMonthlyAndQuarterlyAsync_PIPoint_NotNull_FindPIPoint()
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

     var pitag = "AR.AR1.P-852A.Equipment_Offline_Status_AMH_CALC_OUTPUT";
     PIServer server = (PIServer)FormatterServices.GetSafeUninitializedObject(typeof(PIServer));

     fackserver.Add(server);
     _serverHelperMoke.Setup(x => x.ConnectedPIServers).Returns(fackserver);

     PIPoint fackpipoint = (PIPoint)FormatterServices.GetSafeUninitializedObject(typeof(PIPoint));
     _pIPointWrapper.Setup(x => x.FindPIPoint(server, pitag)).Returns(fackpipoint);

     AssetsService assetsService = new AssetsService(_assetsRepositoryMock.Object, _serverHelperMoke.Object, _pIPointWrapper.Object);
     //private method 
     var methodInfo = typeof(AssetsService).GetMethod("GetSummariesAsync", BindingFlags.NonPublic | BindingFlags.Instance);
     var task = (Task)methodInfo.Invoke(assetsService, new object[] { fackpipoint, request.startTime, request.endTime, "Month", 1 });
     await task.ConfigureAwait(false);
     var resultProp = task.GetType().GetProperty("Result");
     var privatemethodResponse = resultProp?.GetValue(task);
     Assert.IsNotNull(privatemethodResponse);

     //Act
     var result = await _assetsService.GetMonthlyAndQuarterlyAsync(pitag, request.startTime, request.endTime);
     //Assert
     Assert.IsNotNull(result);
 }
 Test method AMH.AFSDK.UnitTests.ServiceTests.AssetServiceTest.GetMonthlyAndQuarterlyAsync_PIPoint_NotNull_FindPIPoint threw exception: 
System.NullReferenceException: Object reference not set to an instance of an object.
