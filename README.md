[Fact]
    public async Task GetCriticalAssetDetails_ReturnsOk_WhenDataExists()
    {
        // Arrange
        var request = _fixture.Create<GetHierarchyInfoRequest>();
        var affiliateCodes = _fixture.CreateMany<int>(3).ToList();
        var expectedResponse = _fixture.CreateMany<GetCriticalAssetDetailsResponse>(2).ToList();

        _mockConfigServices
            .Setup(s => s.GetAffiliateCodeList(request.affiliateId))
            .Returns(affiliateCodes);

        _mockConfigServices
            .Setup(s => s.GetCriticalAssetDetailsAsyncNew(
                request.page!,
                It.IsAny<string>()))
            .ReturnsAsync(expectedResponse);

        // Act
        var result = await _controller.GetCriticalAssetDetails(request);

        // Assert
        var okResult = Assert.IsType<OkObjectResult>(result);
        var actual = Assert.IsType<List<GetCriticalAssetDetailsResponse>>(okResult.Value);
        Assert.Equal(expectedResponse.Count, actual.Count);
    }

    [Fact]
    public async Task GetCriticalAssetDetails_ReturnsUnauthorized_WhenAffiliateListIsNull()
    {
        // Arrange
        var request = _fixture.Create<GetHierarchyInfoRequest>();

        _mockConfigServices
            .Setup(s => s.GetAffiliateCodeList(request.affiliateId))
            .Returns((List<int>?)null);

        // Act
        var result = await _controller.GetCriticalAssetDetails(request);

        // Assert
        Assert.IsType<UnauthorizedResult>(result);
    }

    [Fact]
    public async Task GetCriticalAssetDetails_ReturnsNoContent_WhenResponseIsNull()
    {
        // Arrange
        var request = _fixture.Create<GetHierarchyInfoRequest>();
        var affiliateCodes = _fixture.CreateMany<int>(3).ToList();

        _mockConfigServices
            .Setup(s => s.GetAffiliateCodeList(request.affiliateId))
            .Returns(affiliateCodes);

        _mockConfigServices
            .Setup(s => s.GetCriticalAssetDetailsAsyncNew(
                It.IsAny<string>(),
                It.IsAny<string>()))
            .ReturnsAsync((List<GetCriticalAssetDetailsResponse>?)null);

        // Act
        var result = await _controller.GetCriticalAssetDetails(request);

        // Assert
        Assert.IsType<NoContentResult>(result);
    }

    [Fact]
    public async Task GetCriticalAssetDetails_ThrowsException_ShouldPropagate()
    {
        // Arrange
        var request = _fixture.Create<GetHierarchyInfoRequest>();
        var affiliateCodes = _fixture.CreateMany<int>(2).ToList();

        _mockConfigServices
            .Setup(s => s.GetAffiliateCodeList(request.affiliateId))
            .Returns(affiliateCodes);

        _mockConfigServices
            .Setup(s => s.GetCriticalAssetDetailsAsyncNew(
                It.IsAny<string>(),
                It.IsAny<string>()))
            .ThrowsAsync(new System.Exception("Database error"));

        // Act & Assert
        await Assert.ThrowsAsync<System.Exception>(() => _controller.GetCriticalAssetDetails(request));
    }
