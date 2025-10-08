[Fact]
    public async Task GetAffiliateList_ReturnsOk_WhenDataExists()
    {
        // Arrange
        var expectedResponse = _fixture.CreateMany<AffiliateListResponse>(3).ToList();

        _mockBenchMarkServices
            .Setup(s => s.GetAffiliateList())
            .ReturnsAsync(expectedResponse);

        // Act
        var result = await _controller.GetAffiliateList();

        // Assert
        var okResult = Assert.IsType<OkObjectResult>(result);
        var actual = Assert.IsType<List<AffiliateListResponse>>(okResult.Value);
        Assert.Equal(expectedResponse.Count, actual.Count);
    }

    [Fact]
    public async Task GetAffiliateList_ReturnsNoContent_WhenDataIsNull()
    {
        // Arrange
        _mockBenchMarkServices
            .Setup(s => s.GetAffiliateList())
            .ReturnsAsync((List<AffiliateListResponse>?)null);

        // Act
        var result = await _controller.GetAffiliateList();

        // Assert
        Assert.IsType<NoContentResult>(result);
    }

    [Fact]
    public async Task GetAffiliateList_ThrowsException_ShouldPropagate()
    {
        // Arrange
        _mockBenchMarkServices
            .Setup(s => s.GetAffiliateList())
            .ThrowsAsync(new System.Exception("Database connection failed"));

        // Act & Assert
        await Assert.ThrowsAsync<System.Exception>(() => _controller.GetAffiliateList());
    }
