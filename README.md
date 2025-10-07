    [Fact]
    public async Task GetAffiliatePlantDistribution_ReturnsOk_WhenDataExists()
    {
        // Arrange
        var request = new GetAffiliatePlantDistributionRequest
        {
            affiliateId = 1,
            page = "Dashboard",
            startDate = "2025-01-01",
            endDate = "2025-02-01",
            kpiCode = "KPI001"
        };

        var affiliateCodes = new List<int> { 101, 102 };
        _configServiceMock.Setup(x => x.GetAffiliateCodeList(It.IsAny<int?>()))
            .Returns(affiliateCodes);

        var expectedData = new List<GroupedData>
        {
            new GroupedData { GroupName = "Plant A", Value = 100 },
            new GroupedData { GroupName = "Plant B", Value = 200 }
        };

        _currentServiceMock.Setup(x => x.GetAffiliatesDistributionAsyncNew(
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
