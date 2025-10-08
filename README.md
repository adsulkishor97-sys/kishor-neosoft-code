using Xunit;
using Moq;
using AutoFixture;
using Microsoft.AspNetCore.Mvc;
using System.Collections.Generic;
using System.Threading.Tasks;
using System.Linq;

public class ConfigController_GetCaseHierarchy_Tests
{
    private readonly Mock<IConfigServices> _mockConfigServices;
    private readonly ConfigController _controller;
    private readonly Fixture _fixture;

    public ConfigController_GetCaseHierarchy_Tests()
    {
        _mockConfigServices = new Mock<IConfigServices>();
        _controller = new ConfigController(_mockConfigServices.Object);
        _fixture = new Fixture();
    }

    [Fact]
    public async Task GetCaseHierarchy_ReturnsOk_WhenDataExists()
    {
        // Arrange
        var fakeAffiliateIds = _fixture.CreateMany<int>(3).ToList();
        var fakeAccessDetails = _fixture.Create<CaseHierarchyTokenAccessDetails>();
        var fakeResponse = _fixture.CreateMany<CaseHierarchyResponse>(2).ToList();

        _mockConfigServices.Setup(s => s.GetAffiliateIdList(It.IsAny<int?>()))
                           .Returns(fakeAffiliateIds);
        _mockConfigServices.Setup(s => s.GetCaseHierarchyTokenAccessDetails())
                           .Returns(fakeAccessDetails);
        _mockConfigServices.Setup(s => s.GetCaseHierarchyAsync(It.IsAny<GetCaseHierarchyRequest>(), fakeAccessDetails))
                           .ReturnsAsync(fakeResponse);

        // Act
        var result = await _controller.GetCaseHierarchy();

        // Assert
        var okResult = Assert.IsType<OkObjectResult>(result);
        var actual = Assert.IsType<List<CaseHierarchyResponse>>(okResult.Value);
        Assert.Equal(fakeResponse.Count, actual.Count);
    }

    [Fact]
    public async Task GetCaseHierarchy_ReturnsNoContent_WhenDataIsNull()
    {
        // Arrange
        _mockConfigServices.Setup(s => s.GetAffiliateIdList(It.IsAny<int?>()))
                           .Returns(_fixture.CreateMany<int>(2).ToList());
        _mockConfigServices.Setup(s => s.GetCaseHierarchyTokenAccessDetails())
                           .Returns(_fixture.Create<CaseHierarchyTokenAccessDetails>());
        _mockConfigServices.Setup(s => s.GetCaseHierarchyAsync(It.IsAny<GetCaseHierarchyRequest>(), It.IsAny<CaseHierarchyTokenAccessDetails>()))
                           .ReturnsAsync((List<CaseHierarchyResponse>?)null);

        // Act
        var result = await _controller.GetCaseHierarchy();

        // Assert
        Assert.IsType<NoContentResult>(result);
    }

    [Fact]
    public async Task GetCaseHierarchy_ThrowsException_ShouldPropagate()
    {
        // Arrange
        _mockConfigServices.Setup(s => s.GetAffiliateIdList(It.IsAny<int?>()))
                           .Throws(new System.Exception("DB failure"));

        // Act & Assert
        await Assert.ThrowsAsync<System.Exception>(() => _controller.GetCaseHierarchy());
    }
