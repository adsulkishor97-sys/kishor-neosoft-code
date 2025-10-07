[Fact]
public async Task GetKpiData_ReturnsOk_WhenDataExists()
{
    // Arrange
    var request = new GetKpiDetailsRequest
    {
        affiliateId = 1,
        plantId = "123",
        page = "Dashboard",
        startDate = "2025-01-01",
        endDate = "2025-02-01"
    };

    var affiliateCodes = new List<int> { 101, 102 };
    _mockConfigServices.Setup(s => s.GetAffiliateCodeList(It.IsAny<int?>()))
        .Returns(affiliateCodes);

    var kpiResponse = new List<GetKpiDataResponse>
{
    new GetKpiDataResponse { kpiCode = "KPI01", kpiName = "Efficiency" }
};

    _mockCurrentServices.Setup(s => s.GetKpiDetailsAsync(
        request.page!,
        It.IsAny<string>(),
        request.startDate!,
        request.endDate!,
        request.plantId!
    )).ReturnsAsync(kpiResponse);

    // Act
    var result = await _controller.GetKpiData(request);

    // Assert
    var okResult = Assert.IsType<OkObjectResult>(result);
    var data = Assert.IsAssignableFrom<List<GetKpiDataResponse>>(okResult.Value);
    Assert.Single(data);
    Assert.Equal("KPI01", data[0].kpiCode);
}
[Fact]
public async Task DownloadFromEcmAsync_ReturnsNoContent_WhenFileIsNull()
{
    // Arrange
    var fixture = new Fixture();

    var fileId = fixture.Create<string>();
    var request = Mock.Of<DownloadFromEcmRequest>(r => r.fileID == fileId);

    _mockEcmServices
        .Setup(s => s.DownloadFromEcmAsync(It.IsAny<DownloadFromEcmRequest>()))
        .ReturnsAsync((DownloadFromEcmResponse?)null);

    // Act
    var result = await _controller.DownloadFromEcmAsync(request);

    // Assert
    Assert.NotNull(result);
    Assert.IsType<NoContentResult>(result);
}
