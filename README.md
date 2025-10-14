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

        // ✅ Setup mock HttpContext with a fake user claim
        var claims = new List<Claim> { new Claim("uid", "101") };
        var identity = new ClaimsIdentity(claims, "mock");
        var user = new ClaimsPrincipal(identity);
        _mockHttpContext.Setup(c => c.User).Returns(user);

        _controller.ControllerContext = new ControllerContext
        {
            HttpContext = _mockHttpContext.Object
        };
    }

    // ✅ TEST 1: Should return 200 OK when data exists
    [Fact]
    public async Task GetSearchHistoryBySearchKey_ReturnsOk_WhenDataExists()
    {
        // Arrange
        var request = _fixture.Create<GetSearchHistoryRequest>();
        var expectedData = _fixture.Create<List<string>>();

        _mockSearchHistoryServices
            .Setup(s => s.GetSearchHistoryBySearchKeyAsync(It.IsAny<GetSearchHistoryRequest>(), It.IsAny<int>()))
            .ReturnsAsync(expectedData);

        // Act
        var result = await _controller.GetSearchHistoryBySearchKey(request);

        // Assert
        var okResult = Assert.IsType<OkObjectResult>(result);
        Assert.Equal(expectedData, okResult.Value);
        _mockSearchHistoryServices.Verify(
            s => s.GetSearchHistoryBySearchKeyAsync(It.IsAny<GetSearchHistoryRequest>(), It.IsAny<int>()),
            Times.Once);
    }

    // ✅ TEST 2: Should return 204 NoContent when no data
    [Fact]
    public async Task GetSearchHistoryBySearchKey_ReturnsNoContent_WhenNoData()
    {
        // Arrange
        var request = _fixture.Create<GetSearchHistoryRequest>();
        List<string>? noData = null;

        _mockSearchHistoryServices
            .Setup(s => s.GetSearchHistoryBySearchKeyAsync(It.IsAny<GetSearchHistoryRequest>(), It.IsAny<int>()))
            .ReturnsAsync(noData);

        // Act
        var result = await _controller.GetSearchHistoryBySearchKey(request);

        // Assert
        Assert.IsType<NoContentResult>(result);
    }

    // ✅ TEST 3: Should propagate exceptions if service throws
    [Fact]
    public async Task GetSearchHistoryBySearchKey_ThrowsException_WhenServiceFails()
    {
        // Arrange
        var request = _fixture.Create<GetSearchHistoryRequest>();
        _mockSearchHistoryServices
            .Setup(s => s.GetSearchHistoryBySearchKeyAsync(It.IsAny<GetSearchHistoryRequest>(), It.IsAny<int>()))
            .ThrowsAsync(new System.Exception("Database Error"));

        // Act & Assert
        await Assert.ThrowsAsync<System.Exception>(() =>
            _controller.GetSearchHistoryBySearchKey(request));
    }
}
