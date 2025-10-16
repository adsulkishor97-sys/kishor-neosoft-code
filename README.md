[Fact]
public async Task HandleAffiliateDistributionNonGlobalPageRequest_ShouldReturnGroupedData_WhenValidDataProvided()
{
    // Arrange
    var fixture = new Fixture();

    // Create mock repository
    var mockRepo = new Mock<ICurrentRepository>();

    // Build fake input data using AutoFixture
    var request = fixture.Build<GetAffiliatePlantDistributionRequest>()
                         .With(x => x.affiliateId, fixture.Create<string>())
                         .With(x => x.kpiCode, fixture.Create<string>())
                         .Create();

    var affiliateRequest = fixture.Create<string>();
    var formattedStartDate = fixture.Create<string>();
    var formattedEndDate = fixture.Create<string>();
    var quotedPmCodes = fixture.Create<string>();

    // Mock GetCaseHierarchyAsync to return sample plant data
    var lstPlants = fixture.CreateMany<GetCaseHierarchyResponse>(3).ToList();
    mockRepo.Setup(r => r.GetCaseHierarchyAsync(It.IsAny<string>()))
            .ReturnsAsync(lstPlants);

    // Mock ExecuteBigDataQuery_New for numerator
    var numResult = fixture.CreateMany<KpiNumeratorDenominatorAffiliateDistribution>(3).ToList();
    mockRepo.Setup(r => r.ExecuteBigDataQuery_New<KpiNumeratorDenominatorAffiliateDistribution>(
            It.Is<string>(q => q.Contains("N")), // crude check for "numerator" query
            It.IsAny<Func<IDataReader, KpiNumeratorDenominatorAffiliateDistribution>>()))
        .ReturnsAsync(numResult);

    // Mock ExecuteBigDataQuery_New for denominator
    var denResult = fixture.CreateMany<KpiNumeratorDenominatorAffiliateDistribution>(3).ToList();
    mockRepo.Setup(r => r.ExecuteBigDataQuery_New<KpiNumeratorDenominatorAffiliateDistribution>(
            It.Is<string>(q => q.Contains("D")),
            It.IsAny<Func<IDataReader, KpiNumeratorDenominatorAffiliateDistribution>>()))
        .ReturnsAsync(denResult);

    // Create service instance with mock repo
    var service = (PerformanceSummaryServices)Activator.CreateInstance(
        typeof(PerformanceSummaryServices),
        BindingFlags.Instance | BindingFlags.NonPublic,
        null,
        new object[] { mockRepo.Object },
        null
    )!;

    // Use reflection to get private method
    var method = typeof(PerformanceSummaryServices)
        .GetMethod("HandleAffiliateDistributionNonGlobalPageRequest", BindingFlags.NonPublic | BindingFlags.Instance);

    Assert.NotNull(method);

    // Act: Invoke private async method
    var task = (Task<List<GroupedData>>)method!.Invoke(
        service,
        new object[] { request, affiliateRequest, formattedStartDate, formattedEndDate, quotedPmCodes }
    )!;

    var result = await task;

    // Assert
    Assert.NotNull(result);
    Assert.NotEmpty(result);

    // Ensure structure and data are reasonable
    Assert.All(result, item =>
    {
        Assert.NotNull(item);
        Assert.True(item.Numerator >= 0);
        Assert.True(item.Denominator >= 0);
    });

    // Verify repository calls
    mockRepo.Verify(r => r.GetCaseHierarchyAsync(It.IsAny<string>()), Times.Once);
    mockRepo.Verify(r => r.ExecuteBigDataQuery_New<KpiNumeratorDenominatorAffiliateDistribution>(
        It.IsAny<string>(), It.IsAny<Func<IDataReader, KpiNumeratorDenominatorAffiliateDistribution>>()), Times.AtLeastOnce);
}
