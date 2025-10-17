[Fact]
public async Task HandleAvailabilityKpi_ShouldReturnProcessedData_WhenRepositoryReturnsValidData()
{
    // Arrange
    var item = _fixture.Build<AffiliateDistribution>()
                       .With(x => x.overalN, _fixture.Create<string>())
                       .Create();

    var affiliateRequest1 = _fixture.Create<string>();
    var request = _fixture.Create<GetAffiliatePlantDistributionRequest>();
    var startDate = _fixture.Create<string>();
    var endDate = _fixture.Create<string>();
    var quotedPmCodes = _fixture.Create<string>();

    var fakeData = _fixture.CreateMany<KpiNumeratorDenominatorAffiliateDistribution>(3).ToList();

    _currRepositoryMock
        .Setup(r => r.ExecuteBigDataQuery_New<KpiNumeratorDenominatorAffiliateDistribution>(
            It.IsAny<string>(),
            It.IsAny<Func<DbDataReader, KpiNumeratorDenominatorAffiliateDistribution>>()))
        .ReturnsAsync(fakeData);

    // Use reflection to access the private async method
    var method = typeof(CurrentServices).GetMethod("HandleAvailabilityKpi", BindingFlags.NonPublic | BindingFlags.Static);

    // Act
    var task = (Task<List<GroupedData>>)method!.Invoke(_currServices, new object[]
    {
    item, affiliateRequest1, request, startDate, endDate, quotedPmCodes
    })!;
    var result = await task;

    // Assert
    Assert.NotNull(result);
    Assert.NotEmpty(result);
    _currRepositoryMock.Verify(r => r.ExecuteBigDataQuery_New<KpiNumeratorDenominatorAffiliateDistribution>(
        It.IsAny<string>(),
        It.IsAny<Func<DbDataReader, KpiNumeratorDenominatorAffiliateDistribution>>()), Times.Once);
}
System.NullReferenceException : Object reference not set to an instance of an object.
