    [Fact]
    public void GetCaseHierarchyTokenAccessDetails_ShouldReturn_NonAdminAccess_WithDecryptedIds()
    {
        // Arrange
        var jwtSettings = new JwtSettings { SecretKey = "test-secret-key" };

        var crypt = new CryptographyHelper(jwtSettings);
        var encryptedAffiliate = crypt.AesGcmEncrypt("101");
        var encryptedPlant = crypt.AesGcmEncrypt("501");

        var claims = new List<Claim>
{
    new Claim("affiliateID", encryptedAffiliate),
    new Claim("plantID", encryptedPlant)
};

        var token = new JwtSecurityToken(claims: claims);
        var tokenString = new JwtSecurityTokenHandler().WriteToken(token);

        var user = new ClaimsPrincipal(new ClaimsIdentity(new[]
        {
    new Claim(ClaimTypes.Role, "user")
}, "mock"));

        var httpContext = new DefaultHttpContext();
        httpContext.User = user;
        httpContext.Request.Headers["Authorization"] = $"Bearer {tokenString}";

        _httpContextAccessor.Setup(x => x.HttpContext).Returns(httpContext);

        var service = new ConfigServices(mockConfigRepository.Object, _jwtSettings.Object, _httpContextAccessor.Object);

        // Act
        var result = service.GetCaseHierarchyTokenAccessDetails();

        // Assert
        Assert.NotNull(result);
        Assert.NotNull(result.tokenAffiliateIds);
        Assert.NotNull(result.tokenPlantIds);
        Assert.Equal("non-admin", result.accessRole);
        Assert.Contains("101", result.tokenAffiliateIds);
        Assert.Contains("501", result.tokenPlantIds);
    }
