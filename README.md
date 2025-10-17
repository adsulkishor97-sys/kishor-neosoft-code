[Fact]
public void GetCaseHierarchyTokenAccessDetails_ShouldReturn_NonAdminWithDecryptedIds()
{
    // Arrange
    var fixture = new Fixture();
    var httpContext = new DefaultHttpContext();

    // Non-admin user
    var user = new ClaimsPrincipal(new ClaimsIdentity());
    httpContext.User = user;

    // Use a real JwtSettings instance
    var jwtSettings = new JwtSettings { SecretKey = "test-secret-key" };
    var crypt = new CryptographyHelper(jwtSettings);

    // Encrypt affiliate and plant IDs
    string encryptedAffiliate = crypt.AesGcmEncrypt("101");
    string encryptedPlant = crypt.AesGcmEncrypt("501");

    // Create a simple JWT token with affiliateID and plantID claims
    string headerJson = "{\"alg\":\"HS256\",\"typ\":\"JWT\"}";
    string payloadJson = $"{{\"affiliateID\":[\"{encryptedAffiliate}\"],\"plantID\":[\"{encryptedPlant}\"]}}";

    string header = Base64UrlEncode(System.Text.Encoding.UTF8.GetBytes(headerJson));
    string payload = Base64UrlEncode(System.Text.Encoding.UTF8.GetBytes(payloadJson));
    string signature = Base64UrlEncode(System.Text.Encoding.UTF8.GetBytes(fixture.Create<string>()));

    string validJwt = $"{header}.{payload}.{signature}";
    httpContext.Request.Headers.Authorization = $"Bearer {validJwt}";

    // Mock HttpContext
    _httpContextAccessor.Setup(x => x.HttpContext).Returns(httpContext);

    // Use real JwtSettings in service
    var service = new ConfigServices(mockConfigRepository.Object, jwtSettings, _httpContextAccessor.Object);

    // Act
    var result = service.GetCaseHierarchyTokenAccessDetails();

    // Assert
    Assert.NotNull(result);
    Assert.Equal("non-admin", result.accessRole);
    Assert.NotNull(result.tokenAffiliateIds);
    Assert.Single(result.tokenAffiliateIds);
    Assert.Equal("101", result.tokenAffiliateIds[0]);
    Assert.NotNull(result.tokenPlantIds);
    Assert.Single(result.tokenPlantIds);
    Assert.Equal("501", result.tokenPlantIds[0]);
}
