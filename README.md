[Fact]
    public async Task GetAssetList_ReturnsOk_WhenDataExists()
    {
        // Arrange
        var request = _fixture.Create<GetAssetListRequest>();
        var expectedResponse = _fixture.CreateMany<GetAssetListResponse>(3).ToList();

        _mockBenchmarkServices
            .Setup(s => s.GetAssetList(It.IsAny<GetAssetListRequest>()))
            .ReturnsAsync(expectedResponse);

        // Act
        var result = await _controller.GetAsseList(request);

        // Assert
        var okResult = Assert.IsType<OkObjectResult>(result);
        var actual = Assert.IsType<List<GetAssetListResponse>>(okResult.Value);
        Assert.Equal(expectedResponse.Count, actual.Count);
    }

    [Fact]
    public async Task GetAssetList_ThrowsException_ShouldPropagate()
    {
        // Arrange
        var request = _fixture.Create<GetAssetListRequest>();
        var exceptionMessage = _fixture.Create<string>();

        _mockBenchmarkServices
            .Setup(s => s.GetAssetList(It.IsAny<GetAssetListRequest>()))
            .ThrowsAsync(new System.Exception(exceptionMessage));

        // Act & Assert
        await Assert.ThrowsAsync<System.Exception>(() => _controller.GetAsseList(request));
    }
