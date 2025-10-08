[Fact]
    public async Task GetPlantTrend_ReturnsOk_WhenDataExists()
    {
        // Arrange
        var request = _fixture.Create<PlantTrendRequest>();
        var expectedResponse = _fixture.CreateMany<PlantTrendResponse>(3).ToList();

        _mockBenchMarkServices
            .Setup(s => s.GetPlantTrend(It.IsAny<PlantTrendRequest>()))
            .ReturnsAsync(expectedResponse);

        // Act
        var result = await _controller.GetPlantTrend(request);

        // Assert
        var okResult = Assert.IsType<OkObjectResult>(result);
        var actual = Assert.IsType<List<PlantTrendResponse>>(okResult.Value);
        Assert.Equal(expectedResponse.Count, actual.Count);
    }

    [Fact]
    public async Task GetPlantTrend_ReturnsNoContent_WhenDataIsNull()
    {
        // Arrange
        var request = _fixture.Create<PlantTrendRequest>();

        _mockBenchMarkServices
            .Setup(s => s.GetPlantTrend(It.IsAny<PlantTrendRequest>()))
            .ReturnsAsync((List<PlantTrendResponse>?)null);

        // Act
        var result = await _controller.GetPlantTrend(request);

        // Assert
        Assert.IsType<NoContentResult>(result);
    }

    [Fact]
    public async Task GetPlantTrend_ThrowsException_ShouldPropagate()
    {
        // Arrange
        var request = _fixture.Create<PlantTrendRequest>();

        _mockBenchMarkServices
            .Setup(s => s.GetPlantTrend(It.IsAny<PlantTrendRequest>()))
            .ThrowsAsync(new System.Exception("DB error"));

        // Act & Assert
        await Assert.ThrowsAsync<System.Exception>(() => _controller.GetPlantTrend(request));
    }
