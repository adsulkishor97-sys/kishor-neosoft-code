[Fact]
    public async Task GetPlantFilters_ReturnsOk_WhenDataExists()
    {
        // Arrange
        var expectedResponse = _fixture.CreateMany<FinalFilterResponse>(3).ToList();

        _mockBenchMarkServices
            .Setup(s => s.GetPlantFilters())
            .ReturnsAsync(expectedResponse);

        // Act
        var result = await _controller.GetPlantFilters();

        // Assert
        var okResult = Assert.IsType<OkObjectResult>(result);
        var actual = Assert.IsType<List<FinalFilterResponse>>(okResult.Value);
        Assert.Equal(expectedResponse.Count, actual.Count);
    }

    [Fact]
    public async Task GetPlantFilters_ReturnsNoContent_WhenDataIsNull()
    {
        // Arrange
        _mockBenchMarkServices
            .Setup(s => s.GetPlantFilters())
            .ReturnsAsync((List<FinalFilterResponse>?)null);

        // Act
        var result = await _controller.GetPlantFilters();

        // Assert
        Assert.IsType<NoContentResult>(result);
    }

    [Fact]
    public async Task GetPlantFilters_ThrowsException_ShouldPropagate()
    {
        // Arrange
        _mockBenchMarkServices
            .Setup(s => s.GetPlantFilters())
            .ThrowsAsync(new System.Exception("Database error"));

        // Act & Assert
        await Assert.ThrowsAsync<System.Exception>(() => _controller.GetPlantFilters());
    }
