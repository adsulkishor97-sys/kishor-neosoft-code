using Xunit;
using Moq;
using AutoFixture;
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using System.Security.Claims;
using System.Collections.Generic;
using System.Threading.Tasks;
using YourNamespace.Controllers;
using YourNamespace.Models;
using YourNamespace.Services;
using YourNamespace.Helpers; // For CommonMethod

public class SearchHistoryControllerTests
{
    private readonly IFixture _fixture;
    private readonly Mock<ISearchHistoryServices> _mockSearchHistoryServices;
    private readonly SearchHistoryController _controller;
    private readonly Mock<HttpContext> _mockHttpContext;

    public SearchHistoryControllerTests()
    {
        _fixture = new Fixture();
        _mockSearchHistoryServices = new Mock<ISearchHistoryServices>();
        _mockHttpContext = new Mock<HttpContext>();

        _controller = new SearchHistoryController(_mockSearchHistoryServices.Object);

        // Mock HttpContext with user claims
        var claims = new List<Claim> { new Claim("uid", "123") };
        var identity = new ClaimsIdentity(claims, "mock");
        var user = new ClaimsPrincipal(identity);
        _mockHttpContext.Setup(c => c.User).Returns(user);
        _controller.ControllerContext = new ControllerContext
        {
            HttpContext = _mockHttpContext.Object
        };
    }

    // ✅ Test 1: Returns Ok when record added successfully
    [Fact]
    public async Task AddSearchHistoryBySearchKey_ReturnsOk_WhenIdIsValid()
    {
        // Arrange
        var request = _fixture.Create<SearchHistoryRequest>();
        var expectedId = _fixture.Create<int?>();

        _mockSearchHistoryServices
            .Setup(s => s.AddSearchHistoryBySearchKeyAsync(It.IsAny<SearchHistoryRequest>(), It.IsAny<int>()))
            .ReturnsAsync(expectedId);

        // Act
        var result = await _controller.AddSearchHistoryBySearchKey(request);

        // Assert
        var okResult = Assert.IsType<OkObjectResult>(result);
        Assert.Equal(expectedId, okResult.Value);
    }

    // ✅ Test 2: Returns NoContent when ID is null or <= 0
    [Fact]
    public async Task AddSearchHistoryBySearchKey_ReturnsNoContent_WhenInvalidId()
    {
        // Arrange
        var request = _fixture.Create<SearchHistoryRequest>();
        int? invalidId = 0;

        _mockSearchHistoryServices
            .Setup(s => s.AddSearchHistoryBySearchKeyAsync(It.IsAny<SearchHistoryRequest>(), It.IsAny<int>()))
            .ReturnsAsync(invalidId);

        // Act
        var result = await _controller.AddSearchHistoryBySearchKey(request);

        // Assert
        Assert.IsType<NoContentResult>(result);
    }

    // ✅ Test 3: Throws exception when service fails
    [Fact]
    public async Task AddSearchHistoryBySearchKey_ThrowsException_ShouldPropagate()
    {
        // Arrange
        var request = _fixture.Create<SearchHistoryRequest>();
        var errorMessage = _fixture.Create<string>();

        _mockSearchHistoryServices
            .Setup(s => s.AddSearchHistoryBySearchKeyAsync(It.IsAny<SearchHistoryRequest>(), It.IsAny<int>()))
            .ThrowsAsync(new System.Exception(errorMessage));

        // Act & Assert
        await Assert.ThrowsAsync<System.Exception>(() => _controller.AddSearchHistoryBySearchKey(request));
    }
}
