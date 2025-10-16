

var request = fixture.Build<GetKpiPerformanceRequest>()
    .With(x => x.startDate, DateTime.UtcNow.AddDays(-30))  // 30 days ago
    .With(x => x.endDate, DateTime.UtcNow)                 // today
    .Create();
