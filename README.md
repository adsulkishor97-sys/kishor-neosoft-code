[Fact]
public void GetCaseHierarchyTokenAccessDetails_ShouldHandleMissingAuthHeader()
{
    // Arrange
    var httpContext = new DefaultHttpContext();
    httpContext.Request.Headers["Authorization"] = "";

    _httpContextAccessorMock.Setup(x => x.HttpContext).Returns(httpContext);

    var service = new YourServiceClass(_httpContextAccessorMock.Object, _jwtSettings);

    // Act & Assert
    Assert.ThrowsAny<Exception>(() => service.GetCaseHierarchyTokenAccessDetails());
}
