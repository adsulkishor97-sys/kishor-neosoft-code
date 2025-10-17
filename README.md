[Fact]
public void GetCaseHierarchyTokenAccessDetails_ShouldReturn_NonAdminAccess_WithDecryptedIds()
{
    // Arrange
    var fixture = new Fixture();

    // Generate random values using AutoFixture
    var jwtSettings = new JwtSettings { SecretKey = fixture.Create<string>() };
    var crypt = new CryptographyHelper(jwtSettings);

    var randomAffiliateId = fixture.Create<int>().ToString();
    var randomPlantId = fixture.Create<int>().ToString();

    var encryptedAffiliate = crypt.AesGcmEncrypt(randomAffiliateId);
    var encryptedPlant = crypt.AesGcmEncrypt(randomPlantId);

    var claims = new List<Claim>
    {
        new Claim("affiliateID", encryptedAffiliate),
        new Claim("plantID", encryptedPlant)
    };

    // Generate JWT dynamically
    var token = new JwtSecurityToken(claims: claims);
    var tokenString = new JwtSecurityTokenHandler().WriteToken(token);

    var randomRole = fixture.Create<string>(); // Non-admin role

    var user = new ClaimsPrincipal(new ClaimsIdentity(new[]
    {
        new Claim(ClaimTypes.Role, randomRole)
    }, "mock"));

    var httpContext = new DefaultHttpContext
    {
        User = user
    };

    // Set Authorization header dynamically
    httpContext.Request.Headers["Authorization"] = $"Bearer {tokenString}";

    _httpContextAccessorMock.Setup(x => x.HttpContext).Returns(httpContext);

    // Create instance of the service under test
    var service = new ConfigServices(_httpContextAccessorMock.Object, jwtSettings);

    // Act
    var result = service.GetCaseHierarchyTokenAccessDetails();

    // Assert
    Assert.NotNull(result);
    Assert.Equal("non-admin", result.accessRole);
    Assert.Contains(randomAffiliateId, result.tokenAffiliateIds);
    Assert.Contains(randomPlantId, result.tokenPlantIds);
}
