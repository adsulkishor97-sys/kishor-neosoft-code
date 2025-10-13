using Xunit;
using Moq;
using AutoFixture;
using Microsoft.AspNetCore.Mvc;
using System.Collections.Generic;
using System.Threading.Tasks;

// ✅ Replace these namespaces with your actual ones
// using YourProject.Controllers;
// using YourProject.Models;
// using YourProject.Services;

public class BenchmarkControllerTests
{
    private readonly Mock<IBenchMarkServices> _mockBenchMarkServices;
    private readonly BenchmarkController _controller;
    private readonly Fixture _fixture;

    public BenchmarkControllerTests()
    {
        _fixture = new Fixture();
        _mockBenchMarkServices = new Mock<IBenchMarkServices>();
        _controller = new BenchmarkController(_mockBenchMarkServices.Object);
    }

    [Fact]
    public async Task GetAffiliateBenchmarkComparision_ReturnsOk_WhenDataExists()
    {
        // Arrange
        var request = _fixture.Create<AffiliateBenchmarkComparisionRequest>();
        var mockResponse = _fixture.CreateMany<AffiliateBenchmarkComparisionResponse>(3).ToList();

        _mockBenchMarkServices
            .Setup(s => s.GetAffiliateBenchmarkComparisionAsync(It.IsAny<AffiliateBenchmarkComparisionRequest>()))
            .ReturnsAsync(mockResponse);

        // Act
        var result = await _controller.GetAffiliateBenchmarkComparision(request);

        // Assert
        var okResult = Assert.IsType<OkObjectResult>(result);
        var returnValue = Assert.IsType<List<AffiliateBenchmarkComparisionResponse>>(okResult.Value);
        Assert.Equal(mockResponse.Count, returnValue.Count);
    }

    [Fact]
    public async Task GetAffiliateBenchmarkComparision_ReturnsNoContent_WhenServiceReturnsNull()
    {
        // Arrange
        var request = _fixture.Create<AffiliateBenchmarkComparisionRequest>();

        _mockBenchMarkServices
            .Setup(s => s.GetAffiliateBenchmarkComparisionAsync(It.IsAny<AffiliateBenchmarkComparisionRequest>()))
            .ReturnsAsync((List<AffiliateBenchmarkComparisionResponse>?)null);

        // Act
        var result = await _controller.GetAffiliateBenchmarkComparision(request);

        // Assert
        Assert.IsType<NoContentResult>(result);
    }

    [Fact]
    public async Task GetAffiliateBenchmarkComparision_ReturnsNoContent_WhenServiceReturnsEmptyList()
    {
        // Arrange
        var request = _fixture.Create<AffiliateBenchmarkComparisionRequest>();
        var emptyResponse = new List<AffiliateBenchmarkComparisionResponse>();

        _mockBenchMarkServices
            .Setup(s => s.GetAffiliateBenchmarkComparisionAsync(It.IsAny<AffiliateBenchmarkComparisionRequest>()))
            .ReturnsAsync(emptyResponse);

        // Act
        var result = await _controller.GetAffiliateBenchmarkComparision(request);

        // Assert
        Assert.IsType<NoContentResult>(result);
    }
Assert.IsType() Failure: Value is not the exact type
Expected: typeof(Microsoft.AspNetCore.Mvc.NoContentResult)
Actual:   typeof(Microsoft.AspNetCore.Mvc.OkObjectResult)
