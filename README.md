 [Fact]
 public void GetCaseHierarchyTokenAccessDetails_ShouldReturn_NonAdminWithDecryptedIds()
 {
     // Arrange
     var fixture = new Fixture();
     var httpContext = new DefaultHttpContext();

     var user = new ClaimsPrincipal(new ClaimsIdentity()); // no roles -> non-admin
     httpContext.User = user;
     var crypt = new CryptographyHelper((JwtSettings)_jwtSettings.Object);
     string encryptedAffiliate = crypt.AesGcmEncrypt("101");
     string encryptedPlant = crypt.AesGcmEncrypt("501");

     string headerJson = "{\"alg\":\"HS256\",\"typ\":\"JWT\"}";
     string payloadJson = $"{{\"affiliateID\":[\"{encryptedAffiliate}\"],\"plantID\":[\"{encryptedPlant}\"]}}";

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
     Assert.NotNull(result.tokenAffiliateIds);
     Assert.Single(result.tokenAffiliateIds);
     Assert.Equal("101", result.tokenAffiliateIds[0]);
     Assert.NotNull(result.tokenPlantIds);
     Assert.Single(result.tokenPlantIds);
     Assert.Equal("501", result.tokenPlantIds[0]);
 }
 System.InvalidCastException : Unable to cast object of type 'Castle.Proxies.IOptions`1Proxy' to type 'AMHDomain.Models.Account.JwtSettings'.
