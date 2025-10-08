[Fact]
    public async Task GetAssetFilters_ReturnsOk_WhenDataExists()
    {
        // Arrange
        var expectedResponse = _fixture.CreateMany<GetAssetFiltersResponse>(3).ToList();

        _mockBenchMarkServices
            .Setup(s => s.GetAssetFilters())
            .ReturnsAsync(expectedResponse);

        // Act
        var result = await _controller.GetAsseFilters();

        // Assert
        var okResult = Assert.IsType<OkObjectResult>(result);
        var actual = Assert.IsType<List<GetAssetFiltersResponse>>(okResult.Value);
        Assert.Equal(expectedResponse.Count, actual.Count);
    }

    [Fact]
    public async Task GetAssetFilters_ReturnsNoContent_WhenDataIsNull()
    {
        // Arrange
        _mockBenchMarkServices
            .Setup(s => s.GetAssetFilters())
            .ReturnsAsync((List<GetAssetFiltersResponse>?)null);

        // Act
        var result = await _controller.GetAsseFilters();

        // Assert
        Assert.IsType<NoContentResult>(result);
    }

    [Fact]
    public async Task GetAssetFilters_ThrowsException_ShouldPropagate()
    {
        // Arrange
        _mockBenchMarkServices
            .Setup(s => s.GetAssetFilters())
            .ThrowsAsync(new System.Exception("Database error"));

        // Act & Assert
        await Assert.ThrowsAsync<System.Exception>(() => _controller.GetAsseFilters());
    }
