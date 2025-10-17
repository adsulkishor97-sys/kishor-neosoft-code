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

    // ✅ Dynamically generate a syntactically valid (but dummy) JWT token
    var header = Convert.ToBase64String(System.Text.Encoding.UTF8.GetBytes(
        $"{{\"alg\":\"{fixture.Create<string>()}\",\"typ\":\"JWT\"}}"
    ));
    var payload = Convert.ToBase64String(System.Text.Encoding.UTF8.GetBytes(
        $"{{\"user\":\"{fixture.Create<string>()}\"}}"
    ));
    var signature = Convert.ToBase64String(System.Text.Encoding.UTF8.GetBytes(fixture.Create<string>()));

    var fakeJwtToken = $"{header}.{payload}.{signature}";
    httpContext.Request.Headers["Authorization"] = $"Bearer {fakeJwtToken}";

    // Mock HttpContextAccessor
    _httpContextAccessor.Setup(x => x.HttpContext).Returns(httpContext);

    // Act
    var result = _mockconfigServices.GetCaseHierarchyTokenAccessDetails();

    // Assert
    Assert.NotNull(result);
    Assert.Equal("admin", result.accessRole);
    Assert.Null(result.tokenAffiliateIds);
    Assert.Null(result.tokenPlantIds);
}
  Microsoft.IdentityModel.Tokens.SecurityTokenMalformedException : IDX12709: CanReadToken() returned false. JWT is not well formed.
  The token needs to be in JWS or JWE Compact Serialization Format. (JWS): 'EncodedHeader.EncodedPayload.EncodedSignature'. (JWE): 'EncodedProtectedHeader.EncodedEncryptedKey.EncodedInitializationVector.EncodedCiphertext.EncodedAuthenticationTag'.

Stack Trace: 
  JwtSecurityTokenHandler.ReadJwtToken(String token)
  ConfigServices.GetCaseHierarchyTokenAccessDetails() line 126
  ConfigTests.GetCaseHierarchyTokenAccessDetails_ShouldReturn_AdminAccess_WhenUserIsAdmin() line 316
  RuntimeMethodHandle.InvokeMethod(Object target, Void** arguments, Signature sig, Boolean isConstructor)
  MethodBaseInvoker.InvokeWithNoArgs(Object obj, BindingFlags invokeAttr)
