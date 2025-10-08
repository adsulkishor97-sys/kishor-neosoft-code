[Fact]
    public async Task GetAffiliateTrend_ReturnsOk_WhenDataExists()
    {
        // Arrange
        var request = _fixture.Create<AffiliateTrendRequest>();
        var expectedResponse = _fixture.CreateMany<AffiliateTrendResponse>(3).ToList();

        _mockBenchMarkServices
            .Setup(s => s.GetAffiliateTrend(It.IsAny<AffiliateTrendRequest>()))
            .ReturnsAsync(expectedResponse);

        // Act
        var result = await _controller.GetAffiliateTrend(request);

        // Assert
        var okResult = Assert.IsType<OkObjectResult>(result);
        var actual = Assert.IsType<List<AffiliateTrendResponse>>(okResult.Value);
        Assert.Equal(expectedResponse.Count, actual.Count);
    }

    [Fact]
    public async Task GetAffiliateTrend_ReturnsNoContent_WhenDataIsNull()
    {
        // Arrange
        var request = _fixture.Create<AffiliateTrendRequest>();

        _mockBenchMarkServices
            .Setup(s => s.GetAffiliateTrend(It.IsAny<AffiliateTrendRequest>()))
            .ReturnsAsync((List<AffiliateTrendResponse>?)null);

        // Act
        var result = await _controller.GetAffiliateTrend(request);

        // Assert
        Assert.IsType<NoContentResult>(result);
    }

    [Fact]
    public async Task GetAffiliateTrend_ThrowsException_ShouldPropagate()
    {
        // Arrange
        var request = _fixture.Create<AffiliateTrendRequest>();

        _mockBenchMarkServices
            .Setup(s => s.GetAffiliateTrend(It.IsAny<AffiliateTrendRequest>()))
            .ThrowsAsync(new System.Exception("Database failure"));

        // Act & Assert
        await Assert.ThrowsAsync<System.Exception>(() => _controller.GetAffiliateTrend(request));
    }
