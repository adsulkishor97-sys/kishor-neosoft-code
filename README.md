[Fact]
    public async Task GetAssetTrend_ReturnsOk_WhenDataExists()
    {
        // Arrange
        var request = _fixture.Create<AssetTrendRequest>();
        var expectedResponse = _fixture.CreateMany<AssetTrendResponse>(3).ToList();

        _mockBenchMarkServices
            .Setup(s => s.GetAssetTrend(It.IsAny<AssetTrendRequest>()))
            .ReturnsAsync(expectedResponse);

        // Act
        var result = await _controller.GetAssetTrend(request);

        // Assert
        var okResult = Assert.IsType<OkObjectResult>(result);
        var actual = Assert.IsType<List<AssetTrendResponse>>(okResult.Value);
        Assert.Equal(expectedResponse.Count, actual.Count);
    }

    [Fact]
    public async Task GetAssetTrend_ReturnsNoContent_WhenDataIsNull()
    {
        // Arrange
        var request = _fixture.Create<AssetTrendRequest>();

        _mockBenchMarkServices
            .Setup(s => s.GetAssetTrend(It.IsAny<AssetTrendRequest>()))
            .ReturnsAsync((List<AssetTrendResponse>?)null);

        // Act
        var result = await _controller.GetAssetTrend(request);

        // Assert
        Assert.IsType<NoContentResult>(result);
    }

    [Fact]
    public async Task GetAssetTrend_ThrowsException_ShouldPropagate()
    {
        // Arrange
        var request = _fixture.Create<AssetTrendRequest>();

        _mockBenchMarkServices
            .Setup(s => s.GetAssetTrend(It.IsAny<AssetTrendRequest>()))
            .ThrowsAsync(new System.Exception("DB error"));

        // Act & Assert
        await Assert.ThrowsAsync<System.Exception>(() => _controller.GetAssetTrend(request));
    }
