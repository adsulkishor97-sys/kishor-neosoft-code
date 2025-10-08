 [Fact]
    public async Task GetAffiliateCriteria_ReturnsOk_WhenDataExists()
    {
        // Arrange
        var request = _fixture.Create<GetBenchmarkCriteriaListRequest>();
        var expectedResponse = _fixture.CreateMany<GetBenchmarkCriteriaListResponse>(2).ToList();

        _mockBenchMarkServices
            .Setup(s => s.GetAffiliateCriteria(It.IsAny<GetBenchmarkCriteriaListRequest>()))
            .ReturnsAsync(expectedResponse);

        // Act
        var result = await _controller.GetAffiliateCriteria(request);

        // Assert
        var okResult = Assert.IsType<OkObjectResult>(result);
        var actualResponse = Assert.IsType<List<GetBenchmarkCriteriaListResponse>>(okResult.Value);
        Assert.Equal(expectedResponse.Count, actualResponse.Count);
    }

    [Fact]
    public async Task GetAffiliateCriteria_ReturnsNoContent_WhenDataIsNull()
    {
        // Arrange
        var request = _fixture.Create<GetBenchmarkCriteriaListRequest>();

        _mockBenchMarkServices
            .Setup(s => s.GetAffiliateCriteria(It.IsAny<GetBenchmarkCriteriaListRequest>()))
            .ReturnsAsync((List<GetBenchmarkCriteriaListResponse>?)null);

        // Act
        var result = await _controller.GetAffiliateCriteria(request);

        // Assert
        Assert.IsType<NoContentResult>(result);
    }

    [Fact]
    public async Task GetAffiliateCriteria_ThrowsException_ShouldPropagate()
    {
        // Arrange
        var request = _fixture.Create<GetBenchmarkCriteriaListRequest>();

        _mockBenchMarkServices
            .Setup(s => s.GetAffiliateCriteria(It.IsAny<GetBenchmarkCriteriaListRequest>()))
            .ThrowsAsync(new System.Exception("Database error"));

        // Act & Assert
        await Assert.ThrowsAsync<System.Exception>(() => _controller.GetAffiliateCriteria(request));
    }
