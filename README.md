 [Fact]
    public async Task GetAssetPerformanceCriteria_ReturnsOk_WhenDataExists()
    {
        // Arrange
        var request = _fixture.Create<GetAssetPerformanceRequest>();
        var expectedResponse = _fixture.CreateMany<AssetPerformanceCriteriaResponse>(3).ToList();

        _mockBenchmarkServices
            .Setup(s => s.GetAssetPerformanceCriteria(It.IsAny<GetAssetPerformanceRequest>()))
            .ReturnsAsync(expectedResponse);

        // Act
        var result = await _controller.GetAssetPerformanceCriteria(request);

        // Assert
        var okResult = Assert.IsType<OkObjectResult>(result);
        var actual = Assert.IsType<List<AssetPerformanceCriteriaResponse>>(okResult.Value);
        Assert.Equal(expectedResponse.Count, actual.Count);
    }

    [Fact]
    public async Task GetAssetPerformanceCriteria_ThrowsException_ShouldPropagate()
    {
        // Arrange
        var request = _fixture.Create<GetAssetPerformanceRequest>();
        var exceptionMessage = _fixture.Create<string>();

        _mockBenchmarkServices
            .Setup(s => s.GetAssetPerformanceCriteria(It.IsAny<GetAssetPerformanceRequest>()))
            .ThrowsAsync(new System.Exception(exceptionMessage));

        // Act & Assert
        await Assert.ThrowsAsync<System.Exception>(() => _controller.GetAssetPerformanceCriteria(request));
    }
