[Fact]
public async Task GetHierarchyInfoAsync_ShouldReturnMappedResponse_WhenValidRequest()
{
    // Arrange
    var fixture = new Fixture();

    var request = fixture.Build<GetHierarchyInfoRequest>()
        .With(x => x.affiliateId, 1)
        .With(x => x.plantId, 10)
        .Create();

    var sqlAffiliateRequest = "1";

    var hierarchySqlResponse = new List<GetHierarchyInfoSqlDBResponse>
    {
        new GetHierarchyInfoSqlDBResponse
        {
            affiliateCount = 5,
            assetClassCount = 4,
            plantCount = 3,
            productCount = 2,
            overAllAsset = 20,
            criticalAsset = 5
        }
    };

    var hierarchyPlants = new List<HierarchyPlantResponse>
    {
        new HierarchyPlantResponse
        {
            plantId = 10,
            affiliateId = 1
        }
    };

    // Mock HttpContext user
    var claims = new List<Claim>
    {
        new Claim(ClaimTypes.Role, PageConstants.ADMIN)
    };

    var identity = new ClaimsIdentity(claims, "TestAuthType");
    var claimsPrincipal = new ClaimsPrincipal(identity);

    var httpContext = new DefaultHttpContext
    {
        User = claimsPrincipal
    };

    _mockHttpContextAccessor.Setup(x => x.HttpContext).Returns(httpContext);

    // Mock repositories
    _mockAssetRepository
        .Setup(x => x.GetCaseHierarchyAsync(It.IsAny<int>()))
        .ReturnsAsync(hierarchyPlants);

    _mockConfigRepository
        .Setup(x => x.GetHierarchyInfoFromSqlDBAsync(It.IsAny<string>(), It.IsAny<string>()))
        .ReturnsAsync(hierarchySqlResponse);

    // Act
    var result = await _service.GetHierarchyInfoAsync(request, sqlAffiliateRequest);

    // Assert
    Assert.NotNull(result);
    Assert.Equal(5, result.affiliates);
    Assert.Equal(4, result.assetClass);
    Assert.Equal(3, result.plants);
    Assert.Equal(2, result.product);
    Assert.Equal(20, result.assets);
    Assert.Equal(5, result.criticalAssets);
}
[Fact]
public async Task GetHierarchyInfoAsync_ShouldReturnEmptyResponse_WhenSqlAffiliateRequestIsNull()
{
    var fixture = new Fixture();
    var request = fixture.Create<GetHierarchyInfoRequest>();

    var result = await _service.GetHierarchyInfoAsync(request, null);

    Assert.NotNull(result);
    Assert.IsType<GetHierarchyInfoResponse>(result);
}
[Fact]
public async Task GetHierarchyInfoAsync_ShouldWork_WhenPlantIdIsNull()
{
    var fixture = new Fixture();

    var request = fixture.Build<GetHierarchyInfoRequest>()
        .With(x => x.plantId, (int?)null)
        .Create();

    var sqlAffiliateRequest = "1";

    _mockAssetRepository
        .Setup(x => x.GetCaseHierarchyAsync(It.IsAny<int>()))
        .ReturnsAsync(new List<HierarchyPlantResponse>());

    _mockConfigRepository
        .Setup(x => x.GetHierarchyInfoFromSqlDBAsync(It.IsAny<string>(), It.IsAny<string>()))
        .ReturnsAsync(new List<GetHierarchyInfoSqlDBResponse>
        {
            new GetHierarchyInfoSqlDBResponse()
        });

    var result = await _service.GetHierarchyInfoAsync(request, sqlAffiliateRequest);

    Assert.NotNull(result);
}
[Fact]
public async Task GetHierarchyInfoAsync_ShouldHandleAffiliateRole()
{
    var fixture = new Fixture();

    var request = fixture.Create<GetHierarchyInfoRequest>();

    var claims = new List<Claim>
    {
        new Claim(ClaimTypes.Role, PageConstants.AFFILIATE)
    };

    var identity = new ClaimsIdentity(claims, "TestAuthType");
    var claimsPrincipal = new ClaimsPrincipal(identity);

    var httpContext = new DefaultHttpContext
    {
        User = claimsPrincipal
    };

    _mockHttpContextAccessor.Setup(x => x.HttpContext).Returns(httpContext);

    _mockAssetRepository
        .Setup(x => x.GetCaseHierarchyAsync(It.IsAny<int>()))
        .ReturnsAsync(new List<HierarchyPlantResponse>());

    _mockConfigRepository
        .Setup(x => x.GetHierarchyInfoFromSqlDBAsync(It.IsAny<string>(), It.IsAny<string>()))
        .ReturnsAsync(new List<GetHierarchyInfoSqlDBResponse>
        {
            new GetHierarchyInfoSqlDBResponse()
        });

    var result = await _service.GetHierarchyInfoAsync(request, "1");

    Assert.NotNull(result);
}
