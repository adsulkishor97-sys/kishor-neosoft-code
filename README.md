[Fact]
public void GetCaseHierarchyTokenAccessDetails_ShouldReturnNonAdminWithEmptyLists_WhenNoClaims()
{
    // Arrange
    var fixture = new Fixture();
    var httpContext = new DefaultHttpContext();
    var user = new ClaimsPrincipal(new ClaimsIdentity());
    httpContext.User = user;

    string headerJson = "{\"alg\":\"HS256\",\"typ\":\"JWT\"}";
    string payloadJson = "{}";

    string header = Base64UrlEncode(System.Text.Encoding.UTF8.GetBytes(headerJson));
    string payload = Base64UrlEncode(System.Text.Encoding.UTF8.GetBytes(payloadJson));
    string signature = Base64UrlEncode(System.Text.Encoding.UTF8.GetBytes(fixture.Create<string>()));

    string validJwt = $"{header}.{payload}.{signature}";
    httpContext.Request.Headers.Authorization = $"Bearer {validJwt}";

    _httpContextAccessor.Setup(x => x.HttpContext).Returns(httpContext);

    // Act
    var result = _mockconfigServices.GetCaseHierarchyTokenAccessDetails();

    // Assert
    Assert.Equal("non-admin", result.accessRole);
    Assert.Empty(result.tokenAffiliateIds);
    Assert.Empty(result.tokenPlantIds);
}
