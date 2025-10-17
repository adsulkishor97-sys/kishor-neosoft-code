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

    // ✅ Generate a valid Base64URL-encoded JWT
    string headerJson = "{\"alg\":\"HS256\",\"typ\":\"JWT\"}";
    string payloadJson = $"{{\"user\":\"{fixture.Create<string>()}\"}}";

    string header = Base64UrlEncode(System.Text.Encoding.UTF8.GetBytes(headerJson));
    string payload = Base64UrlEncode(System.Text.Encoding.UTF8.GetBytes(payloadJson));
    string signature = Base64UrlEncode(System.Text.Encoding.UTF8.GetBytes(fixture.Create<string>()));

    string validJwt = $"{header}.{payload}.{signature}";

    httpContext.Request.Headers["Authorization"] = $"Bearer {validJwt}";

    _httpContextAccessor.Setup(x => x.HttpContext).Returns(httpContext);

    // Act
    var result = _mockconfigServices.GetCaseHierarchyTokenAccessDetails();

    // Assert
    Assert.NotNull(result);
    Assert.Equal("admin", result.accessRole);
    Assert.Null(result.tokenAffiliateIds);
    Assert.Null(result.tokenPlantIds);
}

/// <summary>
/// Helper method to create Base64URL-safe strings compatible with JWTs.
/// </summary>
private static string Base64UrlEncode(byte[] input)
{
    return Convert.ToBase64String(input)
        .TrimEnd('=')            // remove padding
        .Replace('+', '-')       // 62nd char of encoding
        .Replace('/', '_');      // 63rd char of encoding
}
