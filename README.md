[Fact]
public async Task HandleAffiliateDistributionNonGlobalPageRequest_ShouldReturnGroupedData_WithValidInputs()
{
    // Arrange
    var _fixture = new Fixture();

    // Mock the repository
    var mockRepo = new Mock<ICurrentRepository>();

    // Generate input request using AutoFixture
    var request = _fixture.Build<GetAffiliatePlantDistributionRequest>()
                          .With(x => x.affiliateId, _fixture.Create<string>())
                          .With(x => x.kpiCode, _fixture.Create<string>())
                          .Create();

    var affiliateRequest = _fixture.Create<string>();
    var formattedStartDate = _fixture.Create<string>();
    var formattedEndDate = _fixture.Create<string>();
    var quotedPmCodes = _fixture.Create<string>();

    // Mock GetCaseHierarchyAsync to return some plants
    var lstPlants = _fixture.CreateMany<GetCaseHierarchyResponse>(3).ToList();
    mockRepo.Setup(r => r.GetCaseHierarchyAsync(It.IsAny<string>()))
            .ReturnsAsync(lstPlants);

    // Mock ExecuteBigDataQuery_New for numerator
    var numResult = _fixture.CreateMany<KpiNumeratorDenominatorAffiliateDistribution>(3).ToList();
    mockRepo.Setup(r => r.ExecuteBigDataQuery_New<KpiNumeratorDenominatorAffiliateDistribution>(
            It.Is<string>(q => q.Contains("N")),
            It.IsAny<Func<IDataReader, KpiNumeratorDenominatorAffiliateDistribution>>()))
        .ReturnsAsync(numResult);

    // Mock ExecuteBigDataQuery_New for denominator
    var denResult = _fixture.CreateMany<KpiNumeratorDenominatorAffiliateDistribution>(3).ToList();
    mockRepo.Setup(r => r.ExecuteBigDataQuery_New<KpiNumeratorDenominatorAffiliateDistribution>(
            It.Is<string>(q => q.Contains("D")),
            It.IsAny<Func<IDataReader, KpiNumeratorDenominatorAffiliateDistribution>>()))
        .ReturnsAsync(denResult);

    // Create the service instance
    var service = (PerformanceSummaryServices)Activator.CreateInstance(
        typeof(PerformanceSummaryServices),
        BindingFlags.Instance | BindingFlags.NonPublic,
        null,
        new object[] { mockRepo.Object }, // pass repo if only dependency
        null
    )!;

    // Get private method via reflection
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

    // Ensure the returned data is valid
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
