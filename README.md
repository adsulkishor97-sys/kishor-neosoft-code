using Xunit;
using Moq;
using AutoFixture;
using Microsoft.AspNetCore.Mvc;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

public class HierarchyControllerTests
{
    private readonly Mock<IConfigServices> _mockConfigServices;
    private readonly HierarchyController _controller;
    private readonly Fixture _fixture;

    public HierarchyControllerTests()
    {
        _mockConfigServices = new Mock<IConfigServices>();
        _controller = new HierarchyController(_mockConfigServices.Object);
        _fixture = new Fixture();
    }

    [Fact]
    public async Task GetHierarchyInfo_ReturnsOk_WhenDataExists()
    {
        // Arrange
        var request = _fixture.Create<GetHierarchyInfoRequest>();
        var affiliateCodes = _fixture.CreateMany<int>(3).ToList();
        var expectedResponse = _fixture.Create<GetHierarchyInfoResponse>();

        _mockConfigServices
            .Setup(s => s.GetAffiliateCodeList(request.affiliateId))
            .Returns(affiliateCodes);

        _mockConfigServices
            .Setup(s => s.GetHierarchyInfoAsync(
                request.page!,
                It.IsAny<string>(),
                It.IsAny<string>(),
                request.affiliateId!,
                request.plantId))
            .ReturnsAsync(expectedResponse);

        // Act
        var result = await _controller.GetHierarchyInfo(request);

        // Assert
        var okResult = Assert.IsType<OkObjectResult>(result);
        var actual = Assert.IsType<GetHierarchyInfoResponse>(okResult.Value);
        Assert.Equal(expectedResponse.affiliates, actual.affiliates);
    }

    [Fact]
    public async Task GetHierarchyInfo_ReturnsUnauthorized_WhenAffiliateListIsNull()
    {
        // Arrange
        var request = _fixture.Create<GetHierarchyInfoRequest>();

        _mockConfigServices
            .Setup(s => s.GetAffiliateCodeList(request.affiliateId))
            .Returns((List<int>?)null);

        // Act
        var result = await _controller.GetHierarchyInfo(request);

        // Assert
        Assert.IsType<UnauthorizedResult>(result);
    }

    [Fact]
    public async Task GetHierarchyInfo_ReturnsNoContent_WhenResponseIsNull()
    {
        // Arrange
        var request = _fixture.Create<GetHierarchyInfoRequest>();
        var affiliateCodes = _fixture.CreateMany<int>(2).ToList();

        _mockConfigServices
            .Setup(s => s.GetAffiliateCodeList(request.affiliateId))
            .Returns(affiliateCodes);

        _mockConfigServices
            .Setup(s => s.GetHierarchyInfoAsync(
                It.IsAny<string>(),
                It.IsAny<string>(),
                It.IsAny<string>(),
                It.IsAny<int?>(),
                It.IsAny<int?>()))
            .ReturnsAsync((GetHierarchyInfoResponse?)null);

        // Act
        var result = await _controller.GetHierarchyInfo(request);

        // Assert
        Assert.IsType<NoContentResult>(result);
    }

    [Fact]
    public async Task GetHierarchyInfo_ThrowsException_ShouldPropagate()
    {
        // Arrange
        var request = _fixture.Create<GetHierarchyInfoRequest>();
        var affiliateCodes = _fixture.CreateMany<int>(3).ToList();

        _mockConfigServices
            .Setup(s => s.GetAffiliateCodeList(request.affiliateId))
            .Returns(affiliateCodes);

        _mockConfigServices
            .Setup(s => s.GetHierarchyInfoAsync(
                It.IsAny<string>(),
                It.IsAny<string>(),
                It.IsAny<string>(),
                It.IsAny<int?>(),
                It.IsAny<int?>()))
            .ThrowsAsync(new System.Exception("Database error"));

        // Act & Assert
        await Assert.ThrowsAsync<System.Exception>(() => _controller.GetHierarchyInfo(request));
    }
}
