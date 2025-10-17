[Fact]
public void GetCaseHierarchyTokenAccessDetails_ShouldReturn_NonAdminAccess_WithDecryptedIds()
{
    // Arrange
    var fixture = new Fixture();

    // Create JwtSettings with random secret
    var jwtSettings = new JwtSettings { SecretKey = fixture.Create<string>() };
    var crypt = new CryptographyHelper(jwtSettings);

    // Generate random affiliate & plant IDs, then encrypt them
    var randomAffiliateId = fixture.Create<int>().ToString();
    var randomPlantId = fixture.Create<int>().ToString();

    var encryptedAffiliate = crypt.AesGcmEncrypt(randomAffiliateId);
    var encryptedPlant = crypt.AesGcmEncrypt(randomPlantId);

    // Generate claims dynamically
    var claims = new List<Claim>
    {
        new Claim("affiliateID", encryptedAffiliate),
        new Claim("plantID", encryptedPlant)
    };

    // Create JWT with dynamic claims
    var token = new JwtSecurityToken(claims: claims);
    var tokenString = new JwtSecurityTokenHandler().WriteToken(token);

    // Generate a random non-admin role
    var randomRole = fixture.Create<string>();

    var user = new ClaimsPrincipal(new ClaimsIdentity(new[]
    {
        new Claim(ClaimTypes.Role, randomRole)
    }, "mock"));

    var httpContext = new DefaultHttpContext();
    httpContext.User = user;

    // Add Authorization header dynamically
    httpContext.Request.Headers["Authorization"] = $"Bearer {tokenString}";

    _httpContextAccessorMock.Setup(x => x.HttpContext).Returns(httpContext);

    // Create service under test (same as corporate/admin test)
    var service = new ConfigServices(_httpContextAccessorMock.Object, jwtSettings);

    // Act
    var result = service.GetCaseHierarchyTokenAccessDetails();

    // Assert
    Assert.NotNull(result);
    Assert.Equal("non-admin", result.accessRole);
    Assert.Contains(randomAffiliateId, result.tokenAffiliateIds);
    Assert.Contains(randomPlantId, result.tokenPlantIds);
}
