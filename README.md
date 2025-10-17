[Fact]
public async Task HandleAvailabilityKpi_ShouldReturnProcessedData_WhenRepositoryReturnsValidData()
{
    // Arrange
    var fixture = new Fixture();

    var item = fixture.Build<AffiliateDistribution>()
                      .With(x => x.overalN, fixture.Create<string>())
                      .Create();

    var affiliateRequest1 = fixture.Create<string>();
    var request = fixture.Create<GetAffiliatePlantDistributionRequest>();
    var startDate = fixture.Create<string>();
    var endDate = fixture.Create<string>();
    var quotedPmCodes = fixture.Create<string>();

    var fakeData = fixture.CreateMany<KpiNumeratorDenominatorAffiliateDistribution>(3).ToList();

    // ✅ Mock repository
    var repoMock = new Mock<ICurrentRepository>();
    repoMock.Setup(r => r.ExecuteBigDataQuery_New<KpiNumeratorDenominatorAffiliateDistribution>(
        It.IsAny<string>(),
        It.IsAny<Func<DbDataReader, KpiNumeratorDenominatorAffiliateDistribution>>()))
        .ReturnsAsync(fakeData);

    // ✅ Mock or create service instance properly
    var currentServices = new CurrentServices(repoMock.Object /* add other deps if needed */);

    // ✅ Access private instance method
    var method = typeof(CurrentServices).GetMethod(
        "HandleAvailabilityKpi",
        BindingFlags.NonPublic | BindingFlags.Instance);

    Assert.NotNull(method);

    try
    {
        // ✅ Invoke method using reflection
        var taskObj = method!.Invoke(currentServices, new object[]
        {
            item, affiliateRequest1, request, startDate, endDate, quotedPmCodes
        });

        // ✅ Await properly and catch real inner exceptions
        var task = (Task<List<GroupedData>>)taskObj!;
        var result = await task;

        // ✅ Assertions
        Assert.NotNull(result);
        repoMock.Verify(r => r.ExecuteBigDataQuery_New<KpiNumeratorDenominatorAffiliateDistribution>(
            It.IsAny<string>(),
            It.IsAny<Func<DbDataReader, KpiNumeratorDenominatorAffiliateDistribution>>()), Times.Once);
    }
    catch (TargetInvocationException ex)
    {
        // 🔍 Unwrap actual cause to see what’s null
        throw ex.InnerException ?? ex;
    }
}
