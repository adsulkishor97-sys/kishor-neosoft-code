[Fact]
    public async Task GetAffiliatePlantDistribution_ReturnsOk_WhenDataExists()
    {
        // Arrange
        var request = new GetAffiliatePlantDistributionRequest
        {
            affiliateId = 1,
            page = "Dashboard",
            startDate = "2025-01-01",
            endDate = "2025-02-01"
        };

        var affiliateCodes = new List<int> { 11, 12 };
        _configServiceMock.Setup(s => s.GetAffiliateCodeList(It.IsAny<int?>()))
            .Returns(affiliateCodes);

        var expectedData = new List<GroupedData>
        {
            new GroupedData { affiliateId = "1", name = "Plant A", overall = 90 },
            new GroupedData { affiliateId = "2", name = "Plant B", overall = 75 }
        };

        _currentServiceMock.Setup(s => s.GetAffiliatesDistributionAsyncNew(
            It.IsAny<GetAffiliatePlantDistributionRequest>(),
            It.IsAny<string>(),
            It.IsAny<int?>()
        )).ReturnsAsync(expectedData);

        // Act
        var result = await _controller.GetAffiliatePlantDistribution(request);

        // Assert
        var okResult = Assert.IsType<OkObjectResult>(result);
        var data = Assert.IsType<List<GroupedData>>(okResult.Value);
        Assert.Equal(2, data.Count);
    }

    // ✅ 2. NO CONTENT TEST CASE
    [Fact]
    public async Task GetAffiliatePlantDistribution_ReturnsNoContent_WhenNoData()
    {
        // Arrange
        var request = new GetAffiliatePlantDistributionRequest
        {
            affiliateId = 2
        };

        _configServiceMock.Setup(s => s.GetAffiliateCodeList(It.IsAny<int?>()))
            .Returns(new List<int> { 101 });

        _currentServiceMock.Setup(s => s.GetAffiliatesDistributionAsyncNew(
            It.IsAny<GetAffiliatePlantDistributionRequest>(),
            It.IsAny<string>(),
            It.IsAny<int?>()
        )).ReturnsAsync((List<GroupedData>?)null);

        // Act
        var result = await _controller.GetAffiliatePlantDistribution(request);

        // Assert
        Assert.IsType<NoContentResult>(result);
    }

    // ✅ 3. UNAUTHORIZED TEST CASE
    [Fact]
    public async Task GetAffiliatePlantDistribution_ReturnsUnauthorized_WhenAffiliateListIsNull()
    {
        // Arrange
        var request = new GetAffiliatePlantDistributionRequest
        {
            affiliateId = 99
        };

        _configServiceMock.Setup(s => s.GetAffiliateCodeList(It.IsAny<int?>()))
            .Returns((List<int>?)null);

        // Act
        var result = await _controller.GetAffiliatePlantDistribution(request);

        // Assert
        Assert.IsType<UnauthorizedResult>(result);
    }
