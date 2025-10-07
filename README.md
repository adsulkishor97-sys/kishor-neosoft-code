[Fact]
public async Task GetAssetDistribution_ReturnsOk_WhenDataExists()
{
    // Arrange
    var fixture = new Fixture();

    // Generate request dynamically
    var request = fixture.Build<GetAssetDistributionRequest>()
                         .With(r => r.page, "global") // required property
                         .Create();

    // Generate affiliate codes dynamically
    var affiliateCodes = fixture.CreateMany<int>(2).ToList();
    _mockConfigServices.Setup(x => x.GetAffiliateCodeList(request.affiliateId))
                       .Returns(affiliateCodes);

    // Generate expected response dynamically
    var expectedResult = fixture.CreateMany<AssetGroupedData>(2).ToList();

    _mockCurrentServices
        .Setup(x => x.GetAssetDistributionAsync(It.IsAny<GetAssetDistributionRequest>(), It.IsAny<string>()))
        .ReturnsAsync(expectedResult);

    // Act
    var result = await _controller.GetAssetDistribution(request);

    // Assert
    var okResult = Assert.IsType<OkObjectResult>(result);
    var response = Assert.IsAssignableFrom<List<AssetGroupedData>>(okResult.Value);
    Assert.Equal(expectedResult.Count, response.Count);
}
[Fact]
public async Task GetAssetDistribution_ThrowsException_ShouldPropagate()
{
    // Arrange
    var fixture = new Fixture();

    // Generate request dynamically
    var request = fixture.Create<GetAssetDistributionRequest>();

    _mockConfigServices.Setup(x => x.GetAffiliateCodeList(request.affiliateId))
                       .Throws(new Exception("DB connection failed"));

    // Act & Assert
    await Assert.ThrowsAsync<Exception>(() => _controller.GetAssetDistribution(request));
}
