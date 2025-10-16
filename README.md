[Theory]
[InlineData(nameof(KpiConstant.MaintananceExcellence))]
[InlineData(nameof(KpiConstant.CostEffectivness))]
[InlineData(nameof(KpiConstant.AssetPerformance))]
public async Task GetPerformanceSummaryActualDataNew_ShouldSelect_CorrectResponseList(string performanceSummaryConstant)
{
    // Arrange
    var fixture = new Fixture();

    var request = fixture.Build<GetKpiPerformanceRequest>()
        .With(x => x.performanceSummary, typeof(KpiConstant)
            .GetField(performanceSummaryConstant)!
            .GetValue(null)!.ToString())
        .Create();

    var dateTimeFormatRequest = fixture.Build<DateTimeFormatRequest>()
        .With(x => x.startDateyyyymmdd, "20250101")
        .With(x => x.endDateyyyymmdd, "20250131")
        .With(x => x.startDateyyyymm, "202501")
        .With(x => x.endDateyyyymm, "202501")
        .With(x => x.affiliateRequest, "AFF01")
        .Create();

    // Mock service and target private method
    var serviceMock = new Mock<PerformanceSummaryServices>(null!, null!) { CallBase = true };

    // Mock GetPerformanceSummaryNumDenCalc since it’s awaited inside
    var expectedData = fixture.CreateMany<KpiNumeratorDenominatorPerformanceSummary>(3).ToList();
    serviceMock.Protected()
        .Setup<Task<List<KpiNumeratorDenominatorPerformanceSummary>>>(
            "GetPerformanceSummaryNumDenCalc", 
            ItExpr.IsAny<string>(), ItExpr.IsAny<string>())
        .ReturnsAsync(expectedData);

    var method = typeof(PerformanceSummaryServices)
        .GetMethod("GetPerformanceSummaryActualDataNew", BindingFlags.NonPublic | BindingFlags.Instance)!;

    // Act
    var result = await (Task<List<KpiNumeratorDenominatorPerformanceSummary>>)method.Invoke(
        serviceMock.Object, new object[] { request, dateTimeFormatRequest });

    // Assert
    Assert.NotNull(result);
    Assert.NotEmpty(result);
    serviceMock.Protected().Verify(
        "GetPerformanceSummaryNumDenCalc",
        Times.AtLeastOnce(),
        ItExpr.IsAny<string>(), ItExpr.IsAny<string>());
}

// Act
var taskObj = method.Invoke(serviceMock.Object, new object[] { request, dateTimeFormatRequest });
Assert.NotNull(taskObj); // Prevent null reference

var result = await (Task<List<KpiNumeratorDenominatorPerformanceSummary>>)taskObj!;

// Assert
Assert.NotNull(result);
Assert.NotEmpty(result);
serviceMock.Protected().Verify(
    "GetPerformanceSummaryNumDenCalc",
    Times.AtLeastOnce(),
    ItExpr.IsAny<string>(), ItExpr.IsAny<string>());

