[Fact]
public async Task GetKpiData_ReturnsOk_WhenDataExists()
{
    // Arrange
    var fixture = new Fixture();

    // Create request dynamically
    var request = fixture.Build<GetKpiDetailsRequest>()
                         .With(r => r.page, "Dashboard") // If some value is required
                         .Create();

    // Mock affiliate codes
    var affiliateCodes = fixture.CreateMany<int>(2).ToList();
    _mockConfigServices.Setup(s => s.GetAffiliateCodeList(It.IsAny<int?>()))
        .Returns(affiliateCodes);

    // Mock KPI response
    var kpiResponse = fixture.CreateMany<GetKpiDataResponse>(1).ToList();

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
    Assert.Equal(kpiResponse.Count, data.Count);
}
