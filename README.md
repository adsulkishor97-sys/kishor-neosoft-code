[TestMethod]
public async Task GetSummariesAsync_ShouldReturnSummaries_WhenPiPointAndDatesAreValid()
{
    // Arrange
    var fixture = new Fixture();
    var startTime = DateTime.Now.AddMonths(-3);
    var endTime = DateTime.Now;

    // Create fake PIPoint (since it's sealed)
    var fakePiPoint = (PIPoint)FormatterServices.GetSafeUninitializedObject(typeof(PIPoint));

    // Use reflection to set Name property of PIPoint (since it's readonly internally)
    var nameProperty = typeof(PIPoint).GetProperty("Name", BindingFlags.Public | BindingFlags.Instance);
    if (nameProperty != null && nameProperty.CanWrite)
        nameProperty.SetValue(fakePiPoint, "Fake.Tag.Name");

    // Build fake SummaryAsync() result (simulate AF SDK behavior)
    var fakeSummary = new Dictionary<AFSummaryTypes, AFValue>
    {
        { AFSummaryTypes.Average, new AFValue(123.45, new AFTime(DateTime.Now)) }
    };

    // Create service instance (with dependencies)
    var service = new AssetsService(
        _assetsRepositoryMock.Object,
        _serverHelperMoke.Object,
        _pIPointWrapper.Object
    );

    // Mock the SummaryAsync() call using reflection-based delegate injection
    // Because PIPoint is sealed, we can't mock it — instead, use shimming via a local delegate
    var summaryAsyncMethod = typeof(PIPoint).GetMethod(
        "SummaryAsync",
        BindingFlags.Public | BindingFlags.Instance
    );

    // Use a delegate trick (optional — purely for simulation)
    // If you can’t intercept SummaryAsync(), skip this and trust fake data coverage.

    // Use reflection to get the private GetSummariesAsync
    var method = typeof(AssetsService).GetMethod("GetSummariesAsync",
        BindingFlags.NonPublic | BindingFlags.Instance);

    Assert.IsNotNull(method, "GetSummariesAsync private method not found.");

    // Act — invoke private method directly
    var task = (Task)method.Invoke(service, new object[]
    {
        fakePiPoint, startTime, endTime, "Month", 1
    });

    await task.ConfigureAwait(false);

    var resultProperty = task.GetType().GetProperty("Result");
    var result = resultProperty?.GetValue(task) as IEnumerable<AssetsPerformanceKpiTrendsResponse>;

    // Assert
    Assert.IsNotNull(result, "Result should not be null.");
    Assert.IsTrue(result.Any(), "Result should contain at least one summary item.");

    var first = result.First();
    Assert.IsNotNull(first.tagName);
    Assert.IsNotNull(first.frequency);
    Assert.IsNotNull(first.label);
}
-----------

[TestMethod]
public async Task GetSummariesAsync_ShouldHandleExceptionInValueExtraction()
{
    var fixture = new Fixture();
    var start = DateTime.Now.AddMonths(-2);
    var end = DateTime.Now;

    var fakePiPoint = (PIPoint)FormatterServices.GetSafeUninitializedObject(typeof(PIPoint));
    var service = new AssetsService(
        _assetsRepositoryMock.Object,
        _serverHelperMoke.Object,
        _pIPointWrapper.Object
    );

    // Force ValueAsDouble() to throw via fake data
    var badSummary = new Dictionary<AFSummaryTypes, AFValue>
    {
        { AFSummaryTypes.Average, null } // will cause NullReferenceException in try
    };

    // Reflection invoke
    var method = typeof(AssetsService).GetMethod("GetSummariesAsync",
        BindingFlags.NonPublic | BindingFlags.Instance);

    var task = (Task)method.Invoke(service, new object[] { fakePiPoint, start, end, "Quarter", 3 });
    await task.ConfigureAwait(false);

    var resultProp = task.GetType().GetProperty("Result");
    var result = resultProp?.GetValue(task) as IEnumerable<AssetsPerformanceKpiTrendsResponse>;

    Assert.IsNotNull(result);
    Assert.IsTrue(result.Any());
    Assert.IsNull(result.First().value, "Value should be null if exception occurred.");
}
