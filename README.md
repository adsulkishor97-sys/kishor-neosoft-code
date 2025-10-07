[Fact]
public async Task GetKPITooltip_ReturnsOk_WhenDataExists()
{
    // Arrange
    var request = _fixture.Create<KpiInputRequest>();
    var affiliateCodes = _fixture.CreateMany<int>(3).ToList();

    _mockConfigServices.Setup(s => s.GetAffiliateCodeList(It.IsAny<int?>()))
        .Returns(affiliateCodes);

    var mockResponse = _fixture.Build<KpiTooltipResponse>()
                               .With(x => x.labels, _fixture.CreateMany<Label>(2).ToList())
                               .Create();

    _mockCurrentServices.Setup(s => s.GetKPITooltipAsyncNew(
        It.IsAny<KpiInputRequest>(), It.IsAny<string>(), It.IsAny<string>(), It.IsAny<string>()))
        .ReturnsAsync(mockResponse);

    // Act
    var result = await _controller.GetKPITooltip(request);

    // Assert
    var okResult = Assert.IsType<OkObjectResult>(result);
    var value = Assert.IsType<KpiTooltipResponse>(okResult.Value);
    Assert.NotEmpty(value.labels);
}
