using Xunit;
using Moq;
using AutoFixture;
using System.Collections.Generic;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using YourNamespace.Controllers;
using YourNamespace.Models;
using YourNamespace.Services;

public class AffiliateBenchmarkControllerTests
{
    private readonly IFixture _fixture;
    private readonly Mock<IBenchMarkServices> _mockBenchMarkServices;
    private readonly BenchmarkController _controller;

    public AffiliateBenchmarkControllerTests()
    {
        _fixture = new Fixture();
        _mockBenchMarkServices = new Mock<IBenchMarkServices>();
        _controller = new BenchmarkController(_mockBenchMarkServices.Object);
    }

    // ✅ Test 1: Returns Ok when service returns data
    [Fact]
    public async Task GetAffiliateBenchmark_ReturnsOk_WhenDataExists()
    {
        // Arrange
        var request = _fixture.Create<GetAffiliateBenchmarkRequest>();
        var expectedResponse = _fixture.CreateMany<BenchmarkGroupedData>(3).ToList();

        _mockBenchMarkServices
            .Setup(s => s.GetAffiliateBenchmark(It.IsAny<GetAffiliateBenchmarkRequest>()))
            .ReturnsAsync(expectedResponse);

        // Act
        var result = await _controller.GetAffiliateBenchmark(request);

        // Assert
        var okResult = Assert.IsType<OkObjectResult>(result);
        var actual = Assert.IsType<List<BenchmarkGroupedData>>(okResult.Value);
        Assert.Equal(expectedResponse.Count, actual.Count);
    }

    // ✅ Test 2: Returns NoContent when service returns null
    [Fact]
    public async Task GetAffiliateBenchmark_ReturnsNoContent_WhenServiceReturnsNull()
    {
        // Arrange
        var request = _fixture.Create<GetAffiliateBenchmarkRequest>();

        _mockBenchMarkServices
            .Setup(s => s.GetAffiliateBenchmark(It.IsAny<GetAffiliateBenchmarkRequest>()))
            .ReturnsAsync((List<BenchmarkGroupedData>?)null);

        // Act
        var result = await _controller.GetAffiliateBenchmark(request);

        // Assert
        Assert.IsType<NoContentResult>(result);
    }

    // ✅ Test 3: Propagates exception
    [Fact]
    public async Task GetAffiliateBenchmark_ThrowsException_ShouldPropagate()
    {
        // Arrange
        var request = _fixture.Create<GetAffiliateBenchmarkRequest>();
        var exceptionMessage = _fixture.Create<string>();

        _mockBenchMarkServices
            .Setup(s => s.GetAffiliateBenchmark(It.IsAny<GetAffiliateBenchmarkRequest>()))
            .ThrowsAsync(new System.Exception(exceptionMessage));

        // Act & Assert
        await Assert.ThrowsAsync<System.Exception>(() => _controller.GetAffiliateBenchmark(request));
    }
