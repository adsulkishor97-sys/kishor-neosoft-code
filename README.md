    [Fact]
    public async Task GetKPITooltip_ReturnsOk_WhenDataExists()
    {
        // Arrange
        var request = new KpiInputRequest
        {
            affiliateId = 1,
            kpiCode = "KPI001",
            startDate = "2025-01-01",
            endDate = "2025-02-01"
        };

        var affiliateCodes = new List<int> { 1001 };
        _configServiceMock.Setup(s => s.GetAffiliateCodeList(It.IsAny<int?>()))
            .Returns(affiliateCodes);

        var mockResponse = new KpiTooltipResponse
        {
            labels = new List<Label>
            {
                new Label { title = "Total", uom = "kg", value = "100" }
            },
            kpiTooltipDetails = new KpiTooltipDetails()
        };

        _currentServiceMock.Setup(s => s.GetKPITooltipAsyncNew(
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
        Assert.NotNull(value.labels);
    }
    [Fact]
    public async Task GetKPITooltip_ReturnsNoContent_WhenNoData()
    {
        // Arrange
        var request = new KpiInputRequest
        {
            affiliateId = 1,
            startDate = "2025-01-01",
            endDate = "2025-02-01"
        };

        var affiliateCodes = new List<int> { 10 };
        _configServiceMock.Setup(s => s.GetAffiliateCodeList(It.IsAny<int?>()))
            .Returns(affiliateCodes);

        _currentServiceMock.Setup(s => s.GetKPITooltipAsyncNew(
            It.IsAny<KpiInputRequest>(),
            It.IsAny<string>(),
            It.IsAny<string>(),
            It.IsAny<string>()
        )).ReturnsAsync((KpiTooltipResponse?)null);

        // Act
        var result = await _controller.GetKPITooltip(request);

        // Assert
        Assert.IsType<NoContentResult>(result);
    }
    [Fact]
    public async Task GetKPITooltip_ReturnsUnauthorized_WhenAffiliateListIsNull()
    {
        // Arrange
        var request = new KpiInputRequest
        {
            affiliateId = 99,
            startDate = "2025-01-01",
            endDate = "2025-02-01"
        };

        _configServiceMock.Setup(s => s.GetAffiliateCodeList(It.IsAny<int?>()))
            .Returns((List<int>?)null);

        // Act
        var result = await _controller.GetKPITooltip(request);

        // Assert
        Assert.IsType<UnauthorizedResult>(result);
    }
}
public interface IConfigServices
{
    List<int> GetAffiliateCodeList(int? affiliateId);
}

public interface ICurrentServices
{
    Task<KpiTooltipResponse?> GetKPITooltipAsyncNew(
        KpiInputRequest request, string affiliateRequest, string startDate, string endDate);
}
