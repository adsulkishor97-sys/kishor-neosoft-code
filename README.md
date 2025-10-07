[Fact]
public async Task GetKpiData_ReturnsUnauthorized_WhenAffiliateListNull()
{
    // Arrange
    var request = new GetKpiDetailsRequest
    {
        affiliateId = null
    };

    // Return null to simulate missing affiliate codes
    _configServiceMock.Setup(s => s.GetAffiliateCodeList(It.IsAny<int?>()))
        .Returns((List<int>?)null);

    // Avoid null reference exception by adjusting controller logic
    // or by handling mock gracefully (Option A preferred)
    _currentServiceMock.Setup(s => s.GetKpiDetailsAsync(
        It.IsAny<string>(), It.IsAny<string>(), It.IsAny<string>(), It.IsAny<string>(), It.IsAny<string>()
    )).ReturnsAsync((List<GetKpiDataResponse>?)null);

    // Act
    var result = await _controller.GetKpiData(request);

    // Assert
    Assert.IsType<UnauthorizedResult>(result);
}
