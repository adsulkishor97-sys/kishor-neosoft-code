[Fact]
public void GetCaseHierarchyTokenAccessDetails_ShouldReturn_AdminAccess_WhenUserIsCorporate()
{
    // Arrange
    var fixture = new Fixture();

    // Create a valid HttpContext with Corporate role
    var httpContext = new DefaultHttpContext();
    var user = new ClaimsPrincipal(new ClaimsIdentity(new[]
    {
        new Claim(ClaimTypes.Role, Constants.corporate)
    }, "mock"));
    httpContext.User = user;

    // ✅ Generate syntactically valid Base64URL-encoded JWT
    string headerJson = "{\"alg\":\"HS256\",\"typ\":\"JWT\"}";
    string payloadJson = $"{{\"user\":\"{fixture.Create<string>()}\"}}";

    string header = Base64UrlEncode(System.Text.Encoding.UTF8.GetBytes(headerJson));
    string payload = Base64UrlEncode(System.Text.Encoding.UTF8.GetBytes(payloadJson));
    string signature = Base64UrlEncode(System.Text.Encoding.UTF8.GetBytes(fixture.Create<string>()));

    string validJwt = $"{header}.{payload}.{signature}";

    httpContext.Request.Headers["Authorization"] = $"Bearer {validJwt}";

    _httpContextAccessorMock.Setup(x => x.HttpContext).Returns(httpContext);

    // Act
    var result = _service.GetCaseHierarchyTokenAccessDetails();

    // Assert
    Assert.NotNull(result);
    Assert.Equal("admin", result.accessRole);
    Assert.Null(result.tokenAffiliateIds);
    Assert.Null(result.tokenPlantIds);
}

/// <summary>
/// Helper to produce JWT-compatible Base64URL strings.
/// </summary>
private static string Base64UrlEncode(byte[] input)
{
    return Convert.ToBase64String(input)
        .TrimEnd('=')
        .Replace('+', '-')
        .Replace('/', '_');
}
