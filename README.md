[Fact]
    public async Task GetAssetClass_ReturnsOk_WhenDataExists()
    {
        // Arrange
        var expectedResponse = _fixture.CreateMany<AssetClassResponse>(3).ToList();

        _mockBenchMarkServices
            .Setup(s => s.GetAssetClass())
            .ReturnsAsync(expectedResponse);

        // Act
        var result = await _controller.GetAssetClass();

        // Assert
        var okResult = Assert.IsType<OkObjectResult>(result);
        var actualData = Assert.IsType<List<AssetClassResponse>>(okResult.Value);
        Assert.Equal(expectedResponse.Count, actualData.Count);
    }

    [Fact]
    public async Task GetAssetClass_ReturnsNoContent_WhenDataIsNull()
    {
        // Arrange
        _mockBenchMarkServices
            .Setup(s => s.GetAssetClass())
            .ReturnsAsync((List<AssetClassResponse>?)null);

        // Act
        var result = await _controller.GetAssetClass();

        // Assert
        Assert.IsType<NoContentResult>(result);
    }

    [Fact]
    public async Task GetAssetClass_ThrowsException_ShouldPropagate()
    {
        // Arrange
        _mockBenchMarkServices
            .Setup(s => s.GetAssetClass())
            .ThrowsAsync(new System.Exception("Database failure"));

        // Act & Assert
        await Assert.ThrowsAsync<System.Exception>(() => _controller.GetAssetClass());
    }
