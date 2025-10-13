// ✅ Test 1: Returns Ok when data exists
    [Fact]
    public async Task GetAssetBenchmarkComparision_ReturnsOk_WhenDataExists()
    {
        // Arrange
        var request = _fixture.Create<AssetBenchmarkComparisionRequest>();
        var expectedResponse = _fixture.CreateMany<AssetBenchmarkComparisionResponse>(3).ToList();

        _mockBenchMarkServices
            .Setup(s => s.GetAssetBenchmarkComparisionAsync(It.IsAny<AssetBenchmarkComparisionRequest>()))
            .ReturnsAsync(expectedResponse);

        // Act
        var result = await _controller.GetAssetBenchmarkComparision(request);

        // Assert
        var okResult = Assert.IsType<OkObjectResult>(result);
        var actual = Assert.IsType<List<AssetBenchmarkComparisionResponse>>(okResult.Value);
        Assert.Equal(expectedResponse.Count, actual.Count);
    }

    // ✅ Test 2: Returns NoContent when service returns null
    [Fact]
    public async Task GetAssetBenchmarkComparision_ReturnsNoContent_WhenServiceReturnsNull()
    {
        // Arrange
        var request = _fixture.Create<AssetBenchmarkComparisionRequest>();

        _mockBenchMarkServices
            .Setup(s => s.GetAssetBenchmarkComparisionAsync(It.IsAny<AssetBenchmarkComparisionRequest>()))
            .ReturnsAsync((List<AssetBenchmarkComparisionResponse>?)null);

        // Act
        var result = await _controller.GetAssetBenchmarkComparision(request);

        // Assert
        Assert.IsType<NoContentResult>(result);
    }

    // ✅ Test 3: Propagates exception
    [Fact]
    public async Task GetAssetBenchmarkComparision_ThrowsException_ShouldPropagate()
    {
        // Arrange
        var request = _fixture.Create<AssetBenchmarkComparisionRequest>();
        var exceptionMessage = _fixture.Create<string>();

        _mockBenchMarkServices
            .Setup(s => s.GetAssetBenchmarkComparisionAsync(It.IsAny<AssetBenchmarkComparisionRequest>()))
            .ThrowsAsync(new System.Exception(exceptionMessage));

        // Act & Assert
        await Assert.ThrowsAsync<System.Exception>(() => _controller.GetAssetBenchmarkComparision(request));
    }
