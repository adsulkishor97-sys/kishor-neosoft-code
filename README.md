using Xunit;
using Moq;
using Microsoft.AspNetCore.Mvc;
using System.Threading.Tasks;
using System.Collections.Generic;

public class KpiControllerTests
{
    private readonly Mock<IConfigServices> _mockConfigServices;
    private readonly Mock<ICurrentServices> _mockCurrentServices;
    private readonly YourController _controller; // Replace with your actual controller name

    public KpiControllerTests()
    {
        _mockConfigServices = new Mock<IConfigServices>();
        _mockCurrentServices = new Mock<ICurrentServices>();
        _controller = new YourController(_mockConfigServices.Object, _mockCurrentServices.Object);
    }

    [Fact]
    public async Task GetKpiData_WithValidAffiliateId_ReturnsOkResult()
    {
        // Arrange
        var request = new GetKpiDetailsRequest
        {
            affiliateId = 1,
            plantId = "PLANT001",
            page = "1",
            startDate = "2024-01-01",
            endDate = "2024-12-31"
        };

        var affiliateCodeList = new List<string> { "AFF001", "AFF002", "AFF003" };
        var expectedResult = new { /* Your expected KPI data structure */ };

        _mockConfigServices
            .Setup(x => x.GetAffiliateCodeList(request.affiliateId))
            .Returns(affiliateCodeList);

        _mockCurrentServices
            .Setup(x => x.GetKpiDetailsAsync(
                request.page,
                It.IsAny<string>(),
                request.startDate,
                request.endDate,
                request.plantId))
            .ReturnsAsync(expectedResult);

        // Act
        var result = await _controller.GetKpiData(request);

        // Assert
        var okResult = Assert.IsType<OkObjectResult>(result);
        Assert.NotNull(okResult.Value);
        Assert.Equal(expectedResult, okResult.Value);
    }

    [Fact]
    public async Task GetKpiData_WithEmptyAffiliateCodeList_ReturnsUnauthorized()
    {
        // Arrange
        var request = new GetKpiDetailsRequest
        {
            affiliateId = 1,
            plantId = "PLANT001",
            page = "1",
            startDate = "2024-01-01",
            endDate = "2024-12-31"
        };

        _mockConfigServices
            .Setup(x => x.GetAffiliateCodeList(request.affiliateId))
            .Returns(new List<string>());

        // Act
        var result = await _controller.GetKpiData(request);

        // Assert
        Assert.IsType<UnauthorizedResult>(result);
        _mockCurrentServices.Verify(
            x => x.GetKpiDetailsAsync(It.IsAny<string>(), It.IsAny<string>(), 
                It.IsAny<string>(), It.IsAny<string>(), It.IsAny<string>()), 
            Times.Never);
    }

    [Fact]
    public async Task GetKpiData_WithNullAffiliateCodeList_ReturnsUnauthorized()
    {
        // Arrange
        var request = new GetKpiDetailsRequest
        {
            affiliateId = null,
            plantId = "PLANT001",
            page = "1",
            startDate = "2024-01-01",
            endDate = "2024-12-31"
        };

        _mockConfigServices
            .Setup(x => x.GetAffiliateCodeList(It.IsAny<int?>()))
            .Returns((List<string>)null);

        // Act
        var result = await _controller.GetKpiData(request);

        // Assert
        Assert.IsType<UnauthorizedResult>(result);
    }

    [Fact]
    public async Task GetKpiData_WhenServiceReturnsNull_ReturnsNoContent()
    {
        // Arrange
        var request = new GetKpiDetailsRequest
        {
            affiliateId = 1,
            plantId = "PLANT001",
            page = "1",
            startDate = "2024-01-01",
            endDate = "2024-12-31"
        };

        var affiliateCodeList = new List<string> { "AFF001", "AFF002" };

        _mockConfigServices
            .Setup(x => x.GetAffiliateCodeList(request.affiliateId))
            .Returns(affiliateCodeList);

        _mockCurrentServices
            .Setup(x => x.GetKpiDetailsAsync(
                request.page,
                It.IsAny<string>(),
                request.startDate,
                request.endDate,
                request.plantId))
            .ReturnsAsync((object)null);

        // Act
        var result = await _controller.GetKpiData(request);

        // Assert
        Assert.IsType<NoContentResult>(result);
    }

    [Fact]
    public async Task GetKpiData_VerifyCorrectBigDataAffiliateIdListFormat()
    {
        // Arrange
        var request = new GetKpiDetailsRequest
        {
            affiliateId = 1,
            plantId = "PLANT001",
            page = "1",
            startDate = "2024-01-01",
            endDate = "2024-12-31"
        };

        var affiliateCodeList = new List<string> { "AFF001", "AFF002", "AFF003" };
        var expectedBigDataFormat = $"{{string.Join(\",\", affiliateCodeList.Select(n => $\"'{n}'\"))}}";

        _mockConfigServices
            .Setup(x => x.GetAffiliateCodeList(request.affiliateId))
            .Returns(affiliateCodeList);

        _mockCurrentServices
            .Setup(x => x.GetKpiDetailsAsync(
                request.page,
                It.Is<string>(s => s.Contains("'AFF001'") && s.Contains("'AFF002'") && s.Contains("'AFF003'")),
                request.startDate,
                request.endDate,
                request.plantId))
            .ReturnsAsync(new { success = true });

        // Act
        var result = await _controller.GetKpiData(request);

        // Assert
        _mockCurrentServices.Verify(
            x => x.GetKpiDetailsAsync(
                request.page,
                It.Is<string>(s => s == $"{{{string.Join(",", affiliateCodeList.Select(n => $"'{n}'"))}}}"),
                request.startDate,
                request.endDate,
                request.plantId),
            Times.Once);
    }

    [Theory]
    [InlineData(null, "PLANT001", "1", "2024-01-01", "2024-12-31")]
    [InlineData(1, "", "1", "2024-01-01", "2024-12-31")]
    [InlineData(1, null, "1", "2024-01-01", "2024-12-31")]
    public async Task GetKpiData_WithVariousInputs_HandlesCorrectly(
        int? affiliateId, string plantId, string page, string startDate, string endDate)
    {
        // Arrange
        var request = new GetKpiDetailsRequest
        {
            affiliateId = affiliateId,
            plantId = plantId,
            page = page,
            startDate = startDate,
            endDate = endDate
        };

        var affiliateCodeList = new List<string> { "AFF001" };

        _mockConfigServices
            .Setup(x => x.GetAffiliateCodeList(It.IsAny<int?>()))
            .Returns(affiliateCodeList);

        _mockCurrentServices
            .Setup(x => x.GetKpiDetailsAsync(
                It.IsAny<string>(),
                It.IsAny<string>(),
                It.IsAny<string>(),
                It.IsAny<string>(),
                It.IsAny<string>()))
            .ReturnsAsync(new { data = "test" });

        // Act
        var result = await _controller.GetKpiData(request);

        // Assert
        Assert.NotNull(result);
    }

    [Fact]
    public async Task GetKpiData_ServiceThrowsException_ThrowsException()
    {
        // Arrange
        var request = new GetKpiDetailsRequest
        {
            affiliateId = 1,
            plantId = "PLANT001",
            page = "1",
            startDate = "2024-01-01",
            endDate = "2024-12-31"
        };

        var affiliateCodeList = new List<string> { "AFF001" };

        _mockConfigServices
            .Setup(x => x.GetAffiliateCodeList(request.affiliateId))
            .Returns(affiliateCodeList);

        _mockCurrentServices
            .Setup(x => x.GetKpiDetailsAsync(
                It.IsAny<string>(),
                It.IsAny<string>(),
                It.IsAny<string>(),
                It.IsAny<string>(),
                It.IsAny<string>()))
            .ThrowsAsync(new System.Exception("Database connection failed"));

        // Act & Assert
        await Assert.ThrowsAsync<System.Exception>(() => _controller.GetKpiData(request));
    }

    [Fact]
    public async Task GetKpiData_VerifyAllParametersPassedCorrectly()
    {
        // Arrange
        var request = new GetKpiDetailsRequest
        {
            affiliateId = 1,
            plantId = "PLANT001",
            page = "1",
            startDate = "2024-01-01",
            endDate = "2024-12-31"
        };

        var affiliateCodeList = new List<string> { "AFF001", "AFF002" };

        _mockConfigServices
            .Setup(x => x.GetAffiliateCodeList(request.affiliateId))
            .Returns(affiliateCodeList);

        _mockCurrentServices
            .Setup(x => x.GetKpiDetailsAsync(
                It.IsAny<string>(),
                It.IsAny<string>(),
                It.IsAny<string>(),
                It.IsAny<string>(),
                It.IsAny<string>()))
            .ReturnsAsync(new { success = true });

        // Act
        await _controller.GetKpiData(request);

        // Assert
        _mockCurrentServices.Verify(
            x => x.GetKpiDetailsAsync(
                request.page,
                It.IsAny<string>(),
                request.startDate,
                request.endDate,
                request.plantId),
            Times.Once);

        _mockConfigServices.Verify(
            x => x.GetAffiliateCodeList(request.affiliateId),
            Times.Once);
    }
}
