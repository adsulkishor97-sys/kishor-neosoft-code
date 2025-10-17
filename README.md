[Fact]
public async Task HandleAvailabilityKpi_ShouldReturnProcessedData()
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

    var mockRepo = new Mock<ICurrentRepository>();
    mockRepo.Setup(r => r.ExecuteBigDataQuery_New<KpiNumeratorDenominatorAffiliateDistribution>(
        It.IsAny<string>(),
        It.IsAny<Func<DbDataReader, KpiNumeratorDenominatorAffiliateDistribution>>()))
        .ReturnsAsync(fakeData);

    var currServices = new CurrentServices(mockRepo.Object /*, other deps */);

    // Access private method
    var method = typeof(CurrentServices).GetMethod(
        "HandleAvailabilityKpi",
        BindingFlags.NonPublic | BindingFlags.Instance);
    Assert.NotNull(method);

    // Act
    var task = (Task<List<GroupedData>>)method!.Invoke(currServices, new object[]
    {
        item, affiliateRequest1, request, startDate, endDate, quotedPmCodes
    })!;
    var result = await task;

    // Assert
    Assert.NotNull(result);
    mockRepo.Verify(r => r.ExecuteBigDataQuery_New<KpiNumeratorDenominatorAffiliateDistribution>(
        It.IsAny<string>(),
        It.IsAny<Func<DbDataReader, KpiNumeratorDenominatorAffiliateDistribution>>()), Times.Once);
}
