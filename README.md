using Xunit;
using Moq;
using AutoFixture;
using System.Collections.Generic;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using YourNamespace.Controllers;
using YourNamespace.Models;
using YourNamespace.Services;

public class PerformanceSummaryControllerTests
{
    private readonly IFixture _fixture;
    private readonly Mock<IConfigServices> _mockConfigServices;
    private readonly Mock<IPerformanceSummaryServices> _mockPerformanceSummaryServices;
    private readonly PerformanceSummaryController _controller;

    public PerformanceSummaryControllerTests()
    {
        _fixture = new Fixture();
        _mockConfigServices = new Mock<IConfigServices>();
        _mockPerformanceSummaryServices = new Mock<IPerformanceSummaryServices>();

        _controller = new PerformanceSummaryController(
            _mockConfigServices.Object,
            _mockPerformanceSummaryServices.Object
        );
    }

    [Fact]
    public async Task PerformanceSummary_ReturnsOk_WhenDataExists()
    {
        // Arrange
        var request = _fixture.Create<GetKpiPerformanceRequest>();
        var affiliateCodes = _fixture.CreateMany<int>(3).ToList();
        var expectedResponse = _fixture.Create<KpiPerformanceResponse>();

        _mockConfigServices
            .Setup(s => s.GetAffiliateCodeList(It.IsAny<int?>()))
            .Returns(affiliateCodes);

        _mockPerformanceSummaryServices
            .Setup(s => s.PerformanceSummaryAsync(It.IsAny<GetKpiPerformanceRequest>(), It.IsAny<string>()))
            .ReturnsAsync(expectedResponse);

        // Act
        var result = await _controller.PerformanceSummary(request);

        // Assert
        var okResult = Assert.IsType<OkObjectResult>(result);
        var actual = Assert.IsType<KpiPerformanceResponse>(okResult.Value);
        Assert.Equal(expectedResponse.performanceSummary, actual.performanceSummary);

        _mockConfigServices.Verify(s => s.GetAffiliateCodeList(It.IsAny<int?>()), Times.Once);
        _mockPerformanceSummaryServices.Verify(s => s.PerformanceSummaryAsync(It.IsAny<GetKpiPerformanceRequest>(), It.IsAny<string>()), Times.Once);
    }

    [Fact]
    public async Task PerformanceSummary_ReturnsUnauthorized_WhenAffiliateListIsNull()
    {
        // Arrange
        var request = _fixture.Create<GetKpiPerformanceRequest>();
        _mockConfigServices
            .Setup(s => s.GetAffiliateCodeList(It.IsAny<int?>()))
            .Returns((List<int>?)null);

        // Act
        var result = await _controller.PerformanceSummary(request);

        // Assert
        Assert.IsType<UnauthorizedResult>(result);
    }

    [Fact]
    public async Task PerformanceSummary_ReturnsNoContent_WhenServiceReturnsNull()
    {
        // Arrange
        var request = _fixture.Create<GetKpiPerformanceRequest>();
        var affiliateCodes = _fixture.CreateMany<int>(2).ToList();

        _mockConfigServices
            .Setup(s => s.GetAffiliateCodeList(It.IsAny<int?>()))
            .Returns(affiliateCodes);

        _mockPerformanceSummaryServices
            .Setup(s => s.PerformanceSummaryAsync(It.IsAny<GetKpiPerformanceRequest>(), It.IsAny<string>()))
            .ReturnsAsync((KpiPerformanceResponse?)null);

        // Act
        var result = await _controller.PerformanceSummary(request);

        // Assert
        Assert.IsType<NoContentResult>(result);
    }

    [Fact]
    public async Task PerformanceSummary_ThrowsException_ShouldPropagate()
    {
        // Arrange
        var request = _fixture.Create<GetKpiPerformanceRequest>();
        var affiliateCodes = _fixture.CreateMany<int>(2).ToList();
        var exceptionMessage = _fixture.Create<string>();

        _mockConfigServices
            .Setup(s => s.GetAffiliateCodeList(It.IsAny<int?>()))
            .Returns(affiliateCodes);

        _mockPerformanceSummaryServices
            .Setup(s => s.PerformanceSummaryAsync(It.IsAny<GetKpiPerformanceRequest>(), It.IsAny<string>()))
            .ThrowsAsync(new System.Exception(exceptionMessage));

        // Act & Assert
        await Assert.ThrowsAsync<System.Exception>(() => _controller.PerformanceSummary(request));
    }
}
