[Fact]
public async Task ProcessAffiliatesDistributionAsync_ShouldReturnExpectedResult_WhenPageIsGlobalOrNonGlobal()
{
    // Arrange
    var fixture = new Fixture();

    var request = fixture.Build<GetAffiliatePlantDistributionRequest>()
                         .With(x => x.page, "global") // case 1: can test non-global by changing this
                         .Create();

    var affiliateRequest = fixture.Create<string>();
    var formatedStartDate = fixture.Create<string>();
    var formatedEndDate = fixture.Create<string>();
    var quotedPmCodes = fixture.Create<string>();

    // Mock dependencies for internal method calls
    var mockRepo = new Mock<ICurrentRepository>();
    var mockService = new Mock<ICurrentServices>();

    // create instance of the class under test
    var serviceInstance = new CurrentServices(
        mockRepo.Object,
        Mock.Of<IConfigServices>(),
        Mock.Of<IHttpContextAccessor>(),
        Mock.Of<ILogger<CurrentServices>>()
    );

    // Set up mocks for private method calls
    var expectedResult = fixture.CreateMany<GroupedData>(2).ToList();

    // Use reflection to find private methods and set up for global path
    var handleGlobal = typeof(CurrentServices)
        .GetMethod("HandleAffiliateDistributionGlobalPageRequest", BindingFlags.NonPublic | BindingFlags.Instance);

    var handleNonGlobal = typeof(CurrentServices)
        .GetMethod("HandleAffiliateDistributionNonGlobalPageRequest", BindingFlags.NonPublic | BindingFlags.Instance);

    // Mock both internal private calls to return expectedResult
    if (handleGlobal != null)
        Mock.Get(serviceInstance)
            .Protected()
            .Setup<Task<List<GroupedData>>>("HandleAffiliateDistributionGlobalPageRequest",
                ItExpr.IsAny<GetAffiliatePlantDistributionRequest>(),
                ItExpr.IsAny<string>(),
                ItExpr.IsAny<string>(),
                ItExpr.IsAny<string>(),
                ItExpr.IsAny<string>())
            .ReturnsAsync(expectedResult);

    if (handleNonGlobal != null)
        Mock.Get(serviceInstance)
            .Protected()
            .Setup<Task<List<GroupedData>>>("HandleAffiliateDistributionNonGlobalPageRequest",
                ItExpr.IsAny<GetAffiliatePlantDistributionRequest>(),
                ItExpr.IsAny<string>(),
                ItExpr.IsAny<string>(),
                ItExpr.IsAny<string>(),
                ItExpr.IsAny<string>())
            .ReturnsAsync(expectedResult);

    // Act
    var method = typeof(CurrentServices).GetMethod("ProcessAffiliatesDistributionAsync", BindingFlags.NonPublic | BindingFlags.Instance);
    Assert.NotNull(method);

    var task = (Task)method.Invoke(serviceInstance, new object[] { request, affiliateRequest, formatedStartDate, formatedEndDate, quotedPmCodes })!;
    await task.ConfigureAwait(false);

    var resultProperty = task.GetType().GetProperty("Result");
    var result = resultProperty?.GetValue(task) as List<GroupedData>;

    // Assert
    Assert.NotNull(result);
    Assert.IsType<List<GroupedData>>(result);
}
