// ✅ Test 1: Should return OK when data exists
    [Fact]
    public async Task GetPlantBenchmark_ReturnsOk_WhenDataExists()
    {
        // Arrange
        var request = _fixture.Create<GetPlantBenchmarkRequest>();
        var expectedResponse = _fixture.CreateMany<PlantBenchmarkGroupedData>(3).ToList();

        _mockBenchMarkServices
            .Setup(s => s.GetPlantBenchmark(It.IsAny<GetPlantBenchmarkRequest>()))
            .ReturnsAsync(expectedResponse);

        // Act
        var result = await _controller.GetPlantBenchmark(request);

        // Assert
        var okResult = Assert.IsType<OkObjectResult>(result);
        var actual = Assert.IsType<List<PlantBenchmarkGroupedData>>(okResult.Value);
        Assert.Equal(expectedResponse.Count, actual.Count);
    }

    // ✅ Test 2: Should return NoContent when service returns null
    [Fact]
    public async Task GetPlantBenchmark_ReturnsNoContent_WhenServiceReturnsNull()
    {
        // Arrange
        var request = _fixture.Create<GetPlantBenchmarkRequest>();

        _mockBenchMarkServices
            .Setup(s => s.GetPlantBenchmark(It.IsAny<GetPlantBenchmarkRequest>()))
            .ReturnsAsync((List<PlantBenchmarkGroupedData>?)null);

        // Act
        var result = await _controller.GetPlantBenchmark(request);

        // Assert
        Assert.IsType<NoContentResult>(result);
    }

    // ✅ Test 3: Should propagate exception from service
    [Fact]
    public async Task GetPlantBenchmark_ThrowsException_ShouldPropagate()
    {
        // Arrange
        var request = _fixture.Create<GetPlantBenchmarkRequest>();
        var exceptionMessage = _fixture.Create<string>();

        _mockBenchMarkServices
            .Setup(s => s.GetPlantBenchmark(It.IsAny<GetPlantBenchmarkRequest>()))
            .ThrowsAsync(new System.Exception(exceptionMessage));

        // Act & Assert
        await Assert.ThrowsAsync<System.Exception>(() => _controller.GetPlantBenchmark(request));
    }
