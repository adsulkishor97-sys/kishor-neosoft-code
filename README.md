using Xunit;
using Moq;
using AutoFixture;
using Microsoft.AspNetCore.Mvc;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

public class CaseControllerTests
{
    private readonly Mock<IConfigServices> _mockConfigServices;
    private readonly CaseController _controller;
    private readonly Fixture _fixture;

    public CaseControllerTests()
    {
        _mockConfigServices = new Mock<IConfigServices>();
        _controller = new CaseController(_mockConfigServices.Object);
        _fixture = new Fixture();
    }

    [Fact]
    public async Task GetCaseHierarchy_ReturnsOk_WhenDataExists()
    {
        // Arrange
        var affiliateIds = _fixture.CreateMany<int>(3).ToList();

        var expectedResponse = _fixture.CreateMany<CaseHierarchyResponse>(2).ToList();

        _mockConfigServices.Setup(s => s.GetAffiliateIdList(It.IsAny<int?>()))
            .Returns(affiliateIds);

        _mockConfigServices.Setup(s => s.GetCaseHierarchyAsync(It.IsAny<GetCaseHierarchyRequest>()))
            .ReturnsAsync(expectedResponse);

        // Act
        var result = await _controller.GetCaseHierarchy();

        // Assert
        var okResult = Assert.IsType<OkObjectResult>(result);
        var actual = Assert.IsAssignableFrom<List<CaseHierarchyResponse>>(okResult.Value);
        Assert.Equal(expectedResponse.Count, actual.Count);
    }

    [Fact]
    public async Task GetCaseHierarchy_ReturnsNoContent_WhenDataIsNull()
    {
        // Arrange
        var affiliateIds = _fixture.CreateMany<int>(2).ToList();

        _mockConfigServices.Setup(s => s.GetAffiliateIdList(It.IsAny<int?>()))
            .Returns(affiliateIds);

        _mockConfigServices.Setup(s => s.GetCaseHierarchyAsync(It.IsAny<GetCaseHierarchyRequest>()))
            .ReturnsAsync((List<CaseHierarchyResponse>?)null);

        // Act
        var result = await _controller.GetCaseHierarchy();

        // Assert
        Assert.IsType<NoContentResult>(result);
    }
}
