 // ✅ Test 1: Returns Ok when data exists
    [Fact]
    public async Task GetAffiliateBenchmarkComparision_ReturnsOk_WhenDataExists()
    {
        // Arrange
        var request = _fixture.Create<AffiliateBenchmarkComparisionRequest>();
        var expectedResponse = _fixture.CreateMany<AffiliateBenchmarkComparisionResponse>(3).ToList();

        _mockBenchMarkServices
            .Setup(s => s.GetAffiliateBenchmarkComparisionAsync(It.IsAny<AffiliateBenchmarkComparisionRequest>()))
            .ReturnsAsync(expectedResponse);

        // Act
        var result = await _controller.GetAffiliateBenchmarkComparision(request);

        // Assert
        var okResult = Assert.IsType<OkObjectResult>(result);
        var actual = Assert.IsType<List<AffiliateBenchmarkComparisionResponse>>(okResult.Value);
        Assert.Equal(expectedResponse.Count, actual.Count);
    }

    // ✅ Test 2: Returns NoContent when service returns null or empty
    [Fact]
    public async Task GetAffiliateBenchmarkComparision_ReturnsNoContent_WhenServiceReturnsEmptyOrNull()
    {
        // Arrange
        var request = _fixture.Create<AffiliateBenchmarkComparisionRequest>();

        // Case 1: empty list
        _mockBenchMarkServices
            .Setup(s => s.GetAffiliateBenchmarkComparisionAsync(It.IsAny<AffiliateBenchmarkComparisionRequest>()))
            .ReturnsAsync(new List<AffiliateBenchmarkComparisionResponse>());

        // Act
        var result = await _controller.GetAffiliateBenchmarkComparision(request);

        // Assert
        Assert.IsType<NoContentResult>(result);
    }

    // ✅ Test 3: Propagates exception
    [Fact]
    public async Task GetAffiliateBenchmarkComparision_ThrowsException_ShouldPropagate()
    {
        // Arrange
        var request = _fixture.Create<AffiliateBenchmarkComparisionRequest>();
        var exceptionMessage = _fixture.Create<string>();

        _mockBenchMarkServices
            .Setup(s => s.GetAffiliateBenchmarkComparisionAsync(It.IsAny<AffiliateBenchmarkComparisionRequest>()))
            .ThrowsAsync(new System.Exception(exceptionMessage));

        // Act & Assert
        await Assert.ThrowsAsync<System.Exception>(() => _controller.GetAffiliateBenchmarkComparision(request));
    }
