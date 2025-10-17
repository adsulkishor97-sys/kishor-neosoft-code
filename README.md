[Fact]
public void GetCaseHierarchyTokenAccessDetails_ShouldReturn_AdminAccess_WhenUserIsAdmin()
{
    // Arrange
    var fixture = new Fixture();

    // Mock HttpContext
    var httpContext = new DefaultHttpContext();
    var user = new ClaimsPrincipal(new ClaimsIdentity(new[]
    {
        new Claim(ClaimTypes.Role, Constants.admin)
    }, "mock"));

    httpContext.User = user;
    httpContext.Request.Headers["Authorization"] = $"Bearer {fixture.Create<string>()}";

    _httpContextAccessor.Setup(x => x.HttpContext).Returns(httpContext);

    // ✅ Mock JwtSecurityTokenHandler so it doesn’t throw
    var handlerMock = new Mock<JwtSecurityTokenHandler>();
    handlerMock
        .Setup(h => h.ReadJwtToken(It.IsAny<string>()))
        .Returns(new JwtSecurityToken(claims: new List<Claim>())); // returns empty JWT safely

    // Inject the mock handler into your service (if not DI, use reflection or wrapper)
    // Example: If your service directly creates the handler, we can replace it temporarily.
    // For demo purposes, assume we can inject it like this:
    _mockconfigServices.JwtHandler = handlerMock.Object; // <-- only if service supports this

    // Act
    var result = _mockconfigServices.GetCaseHierarchyTokenAccessDetails();

    // Assert
    Assert.NotNull(result);
    Assert.Equal("admin", result.accessRole);
    Assert.Null(result.tokenAffiliateIds);
    Assert.Null(result.tokenPlantIds);

    // Verify token handler was never called because admin role short-circuits
    handlerMock.Verify(h => h.ReadJwtToken(It.IsAny<string>()), Times.Never);
}
