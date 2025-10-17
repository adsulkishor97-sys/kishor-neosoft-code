private readonly Mock<IHttpContextAccessor> _httpContextAccessorMock;
private readonly JwtSettings _jwtSettings;
private readonly YourServiceClass _service;  // Replace with your actual service class name

public YourServiceTests()
{
    _httpContextAccessorMock = new Mock<IHttpContextAccessor>();
    _jwtSettings = new JwtSettings { SecretKey = "your-secret-key" }; // use dummy key for test
    _service = new YourServiceClass(_httpContextAccessorMock.Object, _jwtSettings);
}

[Fact]
public void GetCaseHierarchyTokenAccessDetails_ShouldReturn_AdminAccess_WhenUserIsAdmin()
{
    // Arrange
    var fixture = new Fixture();

    var httpContext = new DefaultHttpContext();
    var user = new ClaimsPrincipal(new ClaimsIdentity(new[]
    {
        new Claim(ClaimTypes.Role, Constants.admin)
    }, "mock"));

    httpContext.User = user;
    httpContext.Request.Headers["Authorization"] = $"Bearer {fixture.Create<string>()}";

    _httpContextAccessorMock.Setup(x => x.HttpContext).Returns(httpContext);

    // Act
    var result = _service.GetCaseHierarchyTokenAccessDetails();

    // Assert
    Assert.NotNull(result);
    Assert.Equal("admin", result.accessRole);
    Assert.Null(result.tokenAffiliateIds);
    Assert.Null(result.tokenPlantIds);
}
