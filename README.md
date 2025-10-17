using System;
using System.Collections.Generic;
using System.Linq;
using System.Security.Claims;
using System.IdentityModel.Tokens.Jwt;
using Microsoft.AspNetCore.Http;
using Moq;
using AutoFixture;
using Xunit;

public class CaseHierarchyServiceTests
{
    private readonly Mock<IHttpContextAccessor> _httpContextAccessor;
    private readonly Mock<JwtSettings> _jwtSettings;
    private readonly ConfigServices _mockconfigServices;

    public CaseHierarchyServiceTests()
    {
        _httpContextAccessor = new Mock<IHttpContextAccessor>();
        _jwtSettings = new Mock<JwtSettings>();
        _jwtSettings.Setup(x => x.SecretKey).Returns("test-secret-key");

        _mockconfigServices = new ConfigServices(null, _jwtSettings.Object, _httpContextAccessor.Object);
    }

    #region Helper
    private static string Base64UrlEncode(byte[] input) =>
        Convert.ToBase64String(input).TrimEnd('=').Replace('+', '-').Replace('/', '_');

    private HttpContext CreateHttpContext(string token, params string[] roles)
    {
        var claims = roles.Select(r => new Claim(ClaimTypes.Role, r)).ToList();
        var context = new DefaultHttpContext();
        context.User = new ClaimsPrincipal(new ClaimsIdentity(claims, "mock"));
        context.Request.Headers["Authorization"] = $"Bearer {token}";
        return context;
    }
    #endregion

    [Fact]
    public void GetCaseHierarchyTokenAccessDetails_ShouldReturn_AdminAccess_WhenUserIsAdmin()
    {
        var fixture = new Fixture();
        var httpContext = CreateHttpContext(fixture.Create<string>(), Constants.admin);
        _httpContextAccessor.Setup(x => x.HttpContext).Returns(httpContext);

        var result = _mockconfigServices.GetCaseHierarchyTokenAccessDetails();

        Assert.NotNull(result);
        Assert.Equal("admin", result.accessRole);
        Assert.Null(result.tokenAffiliateIds);
        Assert.Null(result.tokenPlantIds);
    }

    [Fact]
    public void GetCaseHierarchyTokenAccessDetails_ShouldReturn_AdminAccess_WhenUserIsCorporate()
    {
        var fixture = new Fixture();
        var httpContext = CreateHttpContext(fixture.Create<string>(), Constants.corporate);
        _httpContextAccessor.Setup(x => x.HttpContext).Returns(httpContext);

        var result = _mockconfigServices.GetCaseHierarchyTokenAccessDetails();

        Assert.NotNull(result);
        Assert.Equal("admin", result.accessRole);
        Assert.Null(result.tokenAffiliateIds);
        Assert.Null(result.tokenPlantIds);
    }

    [Fact]
    public void GetCaseHierarchyTokenAccessDetails_ShouldHandleMissingAuthHeader()
    {
        var httpContext = new DefaultHttpContext();
        httpContext.Request.Headers.Authorization = "";
        _httpContextAccessor.Setup(x => x.HttpContext).Returns(httpContext);

        Assert.ThrowsAny<Exception>(() => _mockconfigServices.GetCaseHierarchyTokenAccessDetails());
    }

    [Fact]
    public void GetCaseHierarchyTokenAccessDetails_ShouldReturn_NonAdminWithDecryptedIds()
    {
        var fixture = new Fixture();
        var httpContext = new DefaultHttpContext();
        httpContext.User = new ClaimsPrincipal(new ClaimsIdentity());

        var crypt = new CryptographyHelper(_jwtSettings.Object);
        string affiliate = crypt.AesGcmEncrypt("101");
        string plant = crypt.AesGcmEncrypt("501");

        string token = CreateJwtToken(new Dictionary<string, string[]> { { "affiliateID", new[] { affiliate } }, { "plantID", new[] { plant } } });
        httpContext.Request.Headers.Authorization = $"Bearer {token}";

        _httpContextAccessor.Setup(x => x.HttpContext).Returns(httpContext);

        var result = _mockconfigServices.GetCaseHierarchyTokenAccessDetails();

        Assert.Equal("non-admin", result.accessRole);
        Assert.Single(result.tokenAffiliateIds);
        Assert.Equal("101", result.tokenAffiliateIds.First());
        Assert.Single(result.tokenPlantIds);
        Assert.Equal("501", result.tokenPlantIds.First());
    }

    [Fact]
    public void GetCaseHierarchyTokenAccessDetails_ShouldReturn_NonAdminWithEmptyLists_WhenClaimsEmpty()
    {
        var fixture = new Fixture();
        var httpContext = new DefaultHttpContext();
        httpContext.User = new ClaimsPrincipal(new ClaimsIdentity());

        string token = CreateJwtToken(new Dictionary<string, string[]> { { "affiliateID", new[] { "" } }, { "plantID", new[] { "" } } });
        httpContext.Request.Headers.Authorization = $"Bearer {token}";

        _httpContextAccessor.Setup(x => x.HttpContext).Returns(httpContext);

        var result = _mockconfigServices.GetCaseHierarchyTokenAccessDetails();

        Assert.Equal("non-admin", result.accessRole);
        Assert.Empty(result.tokenAffiliateIds);
        Assert.Empty(result.tokenPlantIds);
    }

    [Fact]
    public void GetCaseHierarchyTokenAccessDetails_ShouldThrow_WhenHttpContextIsNull()
    {
        _httpContextAccessor.Setup(x => x.HttpContext).Returns((HttpContext)null);
        Assert.Throws<NullReferenceException>(() => _mockconfigServices.GetCaseHierarchyTokenAccessDetails());
    }

    [Fact]
    public void GetCaseHierarchyTokenAccessDetails_ShouldThrow_WhenJwtTokenMalformed()
    {
        var httpContext = new DefaultHttpContext();
        httpContext.User = new ClaimsPrincipal(new ClaimsIdentity());
        httpContext.Request.Headers.Authorization = "Bearer malformed.token.here";
        _httpContextAccessor.Setup(x => x.HttpContext).Returns(httpContext);

        Assert.Throws<ArgumentException>(() => _mockconfigServices.GetCaseHierarchyTokenAccessDetails());
    }

    [Fact]
    public void GetCaseHierarchyTokenAccessDetails_ShouldReturnNonAdminWithEmptyLists_WhenNoClaims()
    {
        var fixture = new Fixture();
        var httpContext = new DefaultHttpContext();
        httpContext.User = new ClaimsPrincipal(new ClaimsIdentity());

        string token = CreateJwtToken(new Dictionary<string, string[]>());
        httpContext.Request.Headers.Authorization = $"Bearer {token}";

        _httpContextAccessor.Setup(x => x.HttpContext).Returns(httpContext);

        var result = _mockconfigServices.GetCaseHierarchyTokenAccessDetails();

        Assert.Equal("non-admin", result.accessRole);
        Assert.Empty(result.tokenAffiliateIds);
        Assert.Empty(result.tokenPlantIds);
    }

    [Fact]
    public void GetCaseHierarchyTokenAccessDetails_ShouldReturnAllDecryptedIds_WhenMultipleClaimsExist()
    {
        var fixture = new Fixture();
        var httpContext = new DefaultHttpContext();
        httpContext.User = new ClaimsPrincipal(new ClaimsIdentity());

        var crypt = new CryptographyHelper(_jwtSettings.Object);
        string affiliate1 = crypt.AesGcmEncrypt("101");
        string affiliate2 = crypt.AesGcmEncrypt("102");
        string plant1 = crypt.AesGcmEncrypt("501");
        string plant2 = crypt.AesGcmEncrypt("502");

        string token = CreateJwtToken(new Dictionary<string, string[]> {
            { "affiliateID", new[] { affiliate1, affiliate2 } },
            { "plantID", new[] { plant1, plant2 } }
        });
        httpContext.Request.Headers.Authorization = $"Bearer {token}";

        _httpContextAccessor.Setup(x => x.HttpContext).Returns(httpContext);

        var result = _mockconfigServices.GetCaseHierarchyTokenAccessDetails();

        Assert.Equal("non-admin", result.accessRole);
        Assert.Equal(2, result.tokenAffiliateIds.Count);
        Assert.Contains("101", result.tokenAffiliateIds);
        Assert.Contains("102", result.tokenAffiliateIds);
        Assert.Equal(2, result.tokenPlantIds.Count);
        Assert.Contains("501", result.tokenPlantIds);
        Assert.Contains("502", result.tokenPlantIds);
    }

    #region Helper - JWT
    private string CreateJwtToken(Dictionary<string, string[]> claimsDict)
    {
        var handler = new JwtSecurityTokenHandler();
        var claims = new List<Claim>();
        foreach (var kv in claimsDict)
            foreach (var val in kv.Value)
                claims.Add(new Claim(kv.Key, val));

        var token = new JwtSecurityToken(claims: claims);
        return handler.WriteToken(token);
    }
    #endregion
}
