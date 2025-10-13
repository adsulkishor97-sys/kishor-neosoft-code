 // ✅ Test 1: Returns OK when service returns data
    [Fact]
    public async Task PerformanceSummaryaffiliate_ReturnsOk_WhenDataExists()
    {
        // Arrange
        var request = _fixture.Create<GetKpiPerformanceRequest>();
        var affiliateCodes = _fixture.CreateMany<int>(3).ToList();
        var expectedResponse = _fixture.Create<KpiPerformanceResponse>();

        _mockConfigServices
            .Setup(s => s.GetAffiliateCodeList(It.IsAny<int?>()))
            .Returns(affiliateCodes);

        _mockPerformanceSummaryServices
            .Setup(s => s.PerformanceSummaryAffiliateAsync(It.IsAny<GetKpiPerformanceRequest>(), It.IsAny<string>()))
            .ReturnsAsync(expectedResponse);

        // Act
        var result = await _controller.PerformanceSummaryaffiliate(request);

        // Assert
        var okResult = Assert.IsType<OkObjectResult>(result);
        var actual = Assert.IsType<KpiPerformanceResponse>(okResult.Value);
        Assert.Equal(expectedResponse.performanceSummary, actual.performanceSummary);

        _mockConfigServices.Verify(s => s.GetAffiliateCodeList(It.IsAny<int?>()), Times.Once);
        _mockPerformanceSummaryServices.Verify(
            s => s.PerformanceSummaryAffiliateAsync(It.IsAny<GetKpiPerformanceRequest>(), It.IsAny<string>()),
            Times.Once
        );
    }

    // ✅ Test 2: Returns Unauthorized when affiliate list is null
    [Fact]
    public async Task PerformanceSummaryaffiliate_ReturnsUnauthorized_WhenAffiliateListIsNull()
    {
        // Arrange
        var request = _fixture.Create<GetKpiPerformanceRequest>();
        _mockConfigServices
            .Setup(s => s.GetAffiliateCodeList(It.IsAny<int?>()))
            .Returns((List<int>?)null);

        // Act
        var result = await _controller.PerformanceSummaryaffiliate(request);

        // Assert
        Assert.IsType<UnauthorizedResult>(result);
    }

    // ✅ Test 3: Returns NoContent when service returns null
    [Fact]
    public async Task PerformanceSummaryaffiliate_ReturnsNoContent_WhenServiceReturnsNull()
    {
        // Arrange
        var request = _fixture.Create<GetKpiPerformanceRequest>();
        var affiliateCodes = _fixture.CreateMany<int>(2).ToList();

        _mockConfigServices
            .Setup(s => s.GetAffiliateCodeList(It.IsAny<int?>()))
            .Returns(affiliateCodes);

        _mockPerformanceSummaryServices
            .Setup(s => s.PerformanceSummaryAffiliateAsync(It.IsAny<GetKpiPerformanceRequest>(), It.IsAny<string>()))
            .ReturnsAsync((KpiPerformanceResponse?)null);

        // Act
        var result = await _controller.PerformanceSummaryaffiliate(request);

        // Assert
        Assert.IsType<NoContentResult>(result);
    }

    // ✅ Test 4: Propagates exception
    [Fact]
    public async Task PerformanceSummaryaffiliate_ThrowsException_ShouldPropagate()
    {
        // Arrange
        var request = _fixture.Create<GetKpiPerformanceRequest>();
        var affiliateCodes = _fixture.CreateMany<int>(2).ToList();
        var exceptionMessage = _fixture.Create<string>();

        _mockConfigServices
            .Setup(s => s.GetAffiliateCodeList(It.IsAny<int?>()))
            .Returns(affiliateCodes);

        _mockPerformanceSummaryServices
            .Setup(s => s.PerformanceSummaryAffiliateAsync(It.IsAny<GetKpiPerformanceRequest>(), It.IsAny<string>()))
            .ThrowsAsync(new System.Exception(exceptionMessage));

        // Act & Assert
        await Assert.ThrowsAsync<System.Exception>(() => _controller.PerformanceSummaryaffiliate(request));
    }
