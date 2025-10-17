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

    // ✅ Use syntactically valid JWT
    httpContext.Request.Headers["Authorization"] = 
        "Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9." +
        "eyJ1c2VyIjoidGVzdCJ9.signature";

    _httpContextAccessor.Setup(x => x.HttpContext).Returns(httpContext);

    // Act
    var result = _mockconfigServices.GetCaseHierarchyTokenAccessDetails();

    // Assert
    Assert.NotNull(result);
    Assert.Equal("admin", result.accessRole);
    Assert.Null(result.tokenAffiliateIds);
    Assert.Null(result.tokenPlantIds);
}
