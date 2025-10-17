[Fact]
public void GetCaseHierarchyTokenAccessDetails_ShouldReturn_NonAdminWithEmptyLists_WhenClaimsEmpty()
{
    // Arrange
    var fixture = new Fixture();
    var httpContext = new DefaultHttpContext();
    var user = new ClaimsPrincipal(new ClaimsIdentity());
    httpContext.User = user;

    string headerJson = "{\"alg\":\"HS256\",\"typ\":\"JWT\"}";
    string payloadJson = "{\"affiliateID\":[\"\"],\"plantID\":[\"\"]}";

    string header = Base64UrlEncode(System.Text.Encoding.UTF8.GetBytes(headerJson));
    string payload = Base64UrlEncode(System.Text.Encoding.UTF8.GetBytes(payloadJson));
    string signature = Base64UrlEncode(System.Text.Encoding.UTF8.GetBytes(fixture.Create<string>()));

    string validJwt = $"{header}.{payload}.{signature}";
    httpContext.Request.Headers.Authorization = $"Bearer {validJwt}";

    _httpContextAccessor.Setup(x => x.HttpContext).Returns(httpContext);

    // Act
    var result = _mockconfigServices.GetCaseHierarchyTokenAccessDetails();

    // Assert
    Assert.NotNull(result);
    Assert.Equal("non-admin", result.accessRole);
    Assert.Empty(result.tokenAffiliateIds);
    Assert.Empty(result.tokenPlantIds);
}
