// ✅ Test 1: Should return OK when service returns data
    [Fact]
    public async Task GetAssetBenchmark_ReturnsOk_WhenDataExists()
    {
        // Arrange
        var request = _fixture.Create<GetAssetBenchmarkRequest>();
        var expectedResponse = _fixture.CreateMany<AssetBenchmarkGroupedData>(3).ToList();

        _mockBenchMarkServices
            .Setup(s => s.GetAssetBenchmark(It.IsAny<GetAssetBenchmarkRequest>()))
            .ReturnsAsync(expectedResponse);

        // Act
        var result = await _controller.GetAssetBenchmark(request);

        // Assert
        var okResult = Assert.IsType<OkObjectResult>(result);
        var actual = Assert.IsType<List<AssetBenchmarkGroupedData>>(okResult.Value);
        Assert.Equal(expectedResponse.Count, actual.Count);
    }

    // ✅ Test 2: Should return NoContent when service returns null
    [Fact]
    public async Task GetAssetBenchmark_ReturnsNoContent_WhenServiceReturnsNull()
    {
        // Arrange
        var request = _fixture.Create<GetAssetBenchmarkRequest>();

        _mockBenchMarkServices
            .Setup(s => s.GetAssetBenchmark(It.IsAny<GetAssetBenchmarkRequest>()))
            .ReturnsAsync((List<AssetBenchmarkGroupedData>?)null);

        // Act
        var result = await _controller.GetAssetBenchmark(request);

        // Assert
        Assert.IsType<NoContentResult>(result);
    }

    // ✅ Test 3: Should return Ok with empty list when no data found (not null)
    [Fact]
    public async Task GetAssetBenchmark_ReturnsOk_WhenServiceReturnsEmptyList()
    {
        // Arrange
        var request = _fixture.Create<GetAssetBenchmarkRequest>();
        var emptyList = new List<AssetBenchmarkGroupedData>();

        _mockBenchMarkServices
            .Setup(s => s.GetAssetBenchmark(It.IsAny<GetAssetBenchmarkRequest>()))
            .ReturnsAsync(emptyList);

        // Act
        var result = await _controller.GetAssetBenchmark(request);

        // Assert
        var okResult = Assert.IsType<OkObjectResult>(result);
        var actual = Assert.IsType<List<AssetBenchmarkGroupedData>>(okResult.Value);
        Assert.Empty(actual);
    }

    // ✅ Test 4: Should propagate exception from service
    [Fact]
    public async Task GetAssetBenchmark_ThrowsException_ShouldPropagate()
    {
        // Arrange
        var request = _fixture.Create<GetAssetBenchmarkRequest>();
        var exceptionMessage = _fixture.Create<string>();

        _mockBenchMarkServices
            .Setup(s => s.GetAssetBenchmark(It.IsAny<GetAssetBenchmarkRequest>()))
            .ThrowsAsync(new System.Exception(exceptionMessage));

        // Act & Assert
        await Assert.ThrowsAsync<System.Exception>(() => _controller.GetAssetBenchmark(request));
    }
