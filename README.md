    [Fact]
    public async Task GetKPITooltip_ReturnsOk_WhenDataExists()
    {
        // Arrange
        var request = new KpiInputRequest
        {
            affiliateId = 1,
            startDate = "2025-01-01",
            endDate = "2025-01-31"
        };

        var affiliateCodes = new List<int> { 101, 102 };
        _configServiceMock.Setup(x => x.GetAffiliateCodeList(It.IsAny<int?>()))
            .Returns(affiliateCodes);

        var expectedResponse = new KpiTooltipResponse
        {
            labels = new List<Label> { new Label { title = "Sample", uom = "kg", value = "10" } },
            kpiTooltipDetails = new KpiTooltipDetails()
        };

        _currentServiceMock.Setup(x => x.GetKPITooltipAsyncNew(
            It.IsAny<KpiInputRequest>(),
            It.IsAny<string>(),
            It.IsAny<string>(),
            It.IsAny<string>()
        )).ReturnsAsync(expectedResponse);

        // Act
        var result = await _controller.GetKPITooltip(request);

        // Assert
        var okResult = Assert.IsType<OkObjectResult>(result);
        var value = Assert.IsType<KpiTooltipResponse>(okResult.Value);
        Assert.NotNull(value.labels);
    }
