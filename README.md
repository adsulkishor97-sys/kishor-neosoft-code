[Fact]
    public async Task GetAssetBenchmark_ReturnsOk_WhenDataExists()
    {
        // Arrange
        var request = _fixture.Create<GetAssetBenchmarkRequest>();
        var expectedResponse = _fixture.CreateMany<AssetBenchmarkGroupedData>(3).ToList();

        _mockBenchmarkServices
            .Setup(s => s.GetAssetBenchmark(It.IsAny<GetAssetBenchmarkRequest>()))
            .ReturnsAsync(expectedResponse);

        // Act
        var result = await _controller.GetAssetBenchmark(request);

        // Assert
        var okResult = Assert.IsType<OkObjectResult>(result);
        var actual = Assert.IsType<List<AssetBenchmarkGroupedData>>(okResult.Value);
        Assert.Equal(expectedResponse.Count, actual.Count);
    }

    [Fact]
    public async Task GetAssetBenchmark_ThrowsException_ShouldPropagate()
    {
        // Arrange
        var request = _fixture.Create<GetAssetBenchmarkRequest>();
        var exceptionMessage = _fixture.Create<string>();

        _mockBenchmarkServices
            .Setup(s => s.GetAssetBenchmark(It.IsAny<GetAssetBenchmarkRequest>()))
            .ThrowsAsync(new System.Exception(exceptionMessage));

        // Act & Assert
        await Assert.ThrowsAsync<System.Exception>(() => _controller.GetAssetBenchmark(request));
    }
