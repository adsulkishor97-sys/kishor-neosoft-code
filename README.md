using Xunit;
using Moq;
using AutoFixture;
using System.Collections.Generic;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using YourNamespace.Controllers;
using YourNamespace.Models;
using YourNamespace.Services;

public class PlantBenchmarkControllerTests
{
    private readonly IFixture _fixture;
    private readonly Mock<IBenchMarkServices> _mockBenchMarkServices;
    private readonly BenchmarkController _controller;

    public PlantBenchmarkControllerTests()
    {
        _fixture = new Fixture();
        _mockBenchMarkServices = new Mock<IBenchMarkServices>();
        _controller = new BenchmarkController(_mockBenchMarkServices.Object);
    }

    // ✅ Test 1: Returns Ok when data exists
    [Fact]
    public async Task GetPlantBenchmarkComparision_ReturnsOk_WhenDataExists()
    {
        // Arrange
        var request = _fixture.Create<PlantBenchmarkComparisionRequest>();
        var expectedResponse = _fixture.CreateMany<PlantBenchmarkComparisionResponse>(3).ToList();

        _mockBenchMarkServices
            .Setup(s => s.GetPlantBenchmarkComparisionAsync(It.IsAny<PlantBenchmarkComparisionRequest>()))
            .ReturnsAsync(expectedResponse);

        // Act
        var result = await _controller.GetPlantBenchmarkComparision(request);

        // Assert
        var okResult = Assert.IsType<OkObjectResult>(result);
        var actual = Assert.IsType<List<PlantBenchmarkComparisionResponse>>(okResult.Value);
        Assert.Equal(expectedResponse.Count, actual.Count);
    }

    // ✅ Test 2: Returns NoContent when service returns empty or null
    [Fact]
    public async Task GetPlantBenchmarkComparision_ReturnsNoContent_WhenServiceReturnsEmptyOrNull()
    {
        // Arrange
        var request = _fixture.Create<PlantBenchmarkComparisionRequest>();

        _mockBenchMarkServices
            .Setup(s => s.GetPlantBenchmarkComparisionAsync(It.IsAny<PlantBenchmarkComparisionRequest>()))
            .ReturnsAsync(new List<PlantBenchmarkComparisionResponse>());

        // Act
        var result = await _controller.GetPlantBenchmarkComparision(request);

        // Assert
        Assert.IsType<NoContentResult>(result);
    }

    // ✅ Test 3: Propagates exception
    [Fact]
    public async Task GetPlantBenchmarkComparision_ThrowsException_ShouldPropagate()
    {
        // Arrange
        var request = _fixture.Create<PlantBenchmarkComparisionRequest>();
        var exceptionMessage = _fixture.Create<string>();

        _mockBenchMarkServices
            .Setup(s => s.GetPlantBenchmarkComparisionAsync(It.IsAny<PlantBenchmarkComparisionRequest>()))
            .ThrowsAsync(new System.Exception(exceptionMessage));

        // Act & Assert
        await Assert.ThrowsAsync<System.Exception>(() => _controller.GetPlantBenchmarkComparision(request));
    }
