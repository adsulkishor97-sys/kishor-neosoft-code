using Xunit;
using Moq;
using AutoFixture;
using Microsoft.AspNetCore.Mvc;
using System.Collections.Generic;
using System.Threading.Tasks;

public class BenchmarkControllerTests
{
    private readonly Mock<IBenchMarkServices> _mockBenchMarkServices;
    private readonly BenchmarkController _controller;
    private readonly Fixture _fixture;

    public BenchmarkControllerTests()
    {
        _mockBenchMarkServices = new Mock<IBenchMarkServices>();
        _controller = new BenchmarkController(_mockBenchMarkServices.Object);
        _fixture = new Fixture();
    }

    [Fact]
    public async Task GetPlantList_ReturnsOk_WhenDataExists()
    {
        // Arrange
        var expectedResponse = _fixture.CreateMany<PlantListResponse>(3).ToList();

        _mockBenchMarkServices
            .Setup(s => s.GetPlantList())
            .ReturnsAsync(expectedResponse);

        // Act
        var result = await _controller.GetPlantList();

        // Assert
        var okResult = Assert.IsType<OkObjectResult>(result);
        var actualResponse = Assert.IsType<List<PlantListResponse>>(okResult.Value);
        Assert.Equal(expectedResponse.Count, actualResponse.Count);
    }

    [Fact]
    public async Task GetPlantList_ReturnsNoContent_WhenDataIsNull()
    {
        // Arrange
        _mockBenchMarkServices
            .Setup(s => s.GetPlantList())
            .ReturnsAsync((List<PlantListResponse>?)null);

        // Act
        var result = await _controller.GetPlantList();

        // Assert
        Assert.IsType<NoContentResult>(result);
    }

    [Fact]
    public async Task GetPlantList_ThrowsException_ShouldPropagate()
    {
        // Arrange
        _mockBenchMarkServices
            .Setup(s => s.GetPlantList())
            .ThrowsAsync(new System.Exception("Database connection failed"));

        // Act & Assert
        await Assert.ThrowsAsync<System.Exception>(() => _controller.GetPlantList());
    }
