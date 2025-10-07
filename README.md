[Fact]
public async Task GetKPITooltip_ReturnsOk_WhenDataExists()
{
    // Arrange
    var fixture = new Fixture();

    // Generate request dynamically
    var request = fixture.Create<KpiInputRequest>();

    // Mock affiliate codes
    var affiliateCodes = fixture.CreateMany<int>(2).ToList();
    _mockConfigServices.Setup(s => s.GetAffiliateCodeList(It.IsAny<int?>()))
        .Returns(affiliateCodes);

    // Mock response dynamically
    var mockResponse = fixture.Build<KpiTooltipResponse>()
                              .With(r => r.labels, fixture.CreateMany<Label>(2).ToList())
                              .With(r => r.kpiTooltipDetails, fixture.Create<KpiTooltipDetails>())
                              .Create();

    _mockCurrentServices.Setup(s => s.GetKPITooltipAsyncNew(
        It.IsAny<KpiInputRequest>(),
        It.IsAny<string>(),
        It.IsAny<string>(),
        It.IsAny<string>()
    )).ReturnsAsync(mockResponse);

    // Act
    var result = await _controller.GetKPITooltip(request);

    // Assert
    var okResult = Assert.IsType<OkObjectResult>(result);
    var value = Assert.IsType<KpiTooltipResponse>(okResult.Value);
    Assert.NotEmpty(value.labels);
}
[Fact]
public async Task GetAffiliatePlantDistribution_ReturnsOk_WhenDataExists()
{
    // Arrange
    var fixture = new Fixture();

    // Generate request dynamically
    var request = fixture.Build<GetAffiliatePlantDistributionRequest>()
                         .With(r => r.page, "Dashboard") // Keep page as required
                         .Create();

    // Mock affiliate codes dynamically
    var affiliateCodes = fixture.CreateMany<int>(2).ToList();
    _mockConfigServices.Setup(s => s.GetAffiliateCodeList(It.IsAny<int?>()))
        .Returns(affiliateCodes);

    // Mock grouped data dynamically
    var expectedData = fixture.CreateMany<GroupedData>(2).ToList();

    _mockCurrentServices.Setup(s => s.GetAffiliatesDistributionAsyncNew(
        It.IsAny<GetAffiliatePlantDistributionRequest>(),
        It.IsAny<string>(),
        It.IsAny<int?>()
    )).ReturnsAsync(expectedData);

    // Act
    var result = await _controller.GetAffiliatePlantDistribution(request);

    // Assert
    var okResult = Assert.IsType<OkObjectResult>(result);
    var data = Assert.IsType<List<GroupedData>>(okResult.Value);
    Assert.Equal(expectedData.Count, data.Count);
}
