using Xunit;
using Moq;
using Microsoft.AspNetCore.Mvc;
using System.Threading.Tasks;
using System.Collections.Generic;
using System.Linq;

public class KpiControllerTests
{
    private readonly Mock<IConfigServices> _mockConfigServices;
    private readonly Mock<ICurrentServices> _mockCurrentServices;
    private readonly KpiController _controller; // Replace with your actual controller name

    public KpiControllerTests()
    {
        _mockConfigServices = new Mock<IConfigServices>();
        _mockCurrentServices = new Mock<ICurrentServices>();
        _controller = new KpiController(_mockConfigServices.Object, _mockCurrentServices.Object);
    }

    [Fact]
    public async Task GetKpiData_WithValidRequest_ReturnsOkResultWithData()
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
        
        var expectedResponse = new GetKpiDataResponse
        {
            plantId = 1,
            kpiName = "Test KPI",
            kpiCode = "KPI001",
            overallActual = 1000,
            overallTargetDisplay = "1200",
            overallTargetMin = 800,
            criticalState = 0,
            overallState = 1,
            criticalActual = 500,
            criticalTargetDisplay = "600",
            criticalTargetMin = 400,
            criticalTargetMax = 700,
            subCategory = "Category A"
        };

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
            .ReturnsAsync(expectedResponse);

        // Act
        var result = await _controller.GetKpiData(request);

        // Assert
        var okResult = Assert.IsType<OkObjectResult>(result);
        var actualResponse = Assert.IsType<GetKpiDataResponse>(okResult.Value);
        
        Assert.Equal(expectedResponse.plantId, actualResponse.plantId);
        Assert.Equal(expectedResponse.kpiName, actualResponse.kpiName);
        Assert.Equal(expectedResponse.kpiCode, actualResponse.kpiCode);
        Assert.Equal(expectedResponse.overallActual, actualResponse.overallActual);
        Assert.Equal(expectedResponse.criticalActual, actualResponse.criticalActual);
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
            x => x.GetKpiDetailsAsync(
                It.IsAny<string>(), 
                It.IsAny<string>(), 
                It.IsAny<string>(), 
                It.IsAny<string>(), 
                It.IsAny<string>()), 
            Times.Never);
    }

    [Fact]
    public async Task GetKpiData_WithNullAffiliateCodeList_ReturnsUnauthorized()
    {
        // Arrange
        var request = new GetKpiDetailsRequest
        {
            affiliateId = 1,
            plantId = "PLANT001",
            page = "1"
        };

        _mockConfigServices
            .Setup(x => x.GetAffiliateCodeList(request.affiliateId))
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
                It.IsAny<string>(),
                It.IsAny<string>(),
                It.IsAny<string>(),
                It.IsAny<string>(),
                It.IsAny<string>()))
            .ReturnsAsync((GetKpiDataResponse)null);

        // Act
        var result = await _controller.GetKpiData(request);

        // Assert
        Assert.IsType<NoContentResult>(result);
    }

    [Fact]
    public async Task GetKpiData_VerifyBigDataAffiliateIdListFormat()
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
        var expectedFormat = $"{{{string.Join(",", affiliateCodeList.Select(n => $"'{n}'"))}}}";

        _mockConfigServices
            .Setup(x => x.GetAffiliateCodeList(request.affiliateId))
            .Returns(affiliateCodeList);

        _mockCurrentServices
            .Setup(x => x.GetKpiDetailsAsync(
                It.IsAny<string>(),
                expectedFormat,
                It.IsAny<string>(),
                It.IsAny<string>(),
                It.IsAny<string>()))
            .ReturnsAsync(new GetKpiDataResponse { plantId = 1 });

        // Act
        await _controller.GetKpiData(request);

        // Assert
        _mockCurrentServices.Verify(
            x => x.GetKpiDetailsAsync(
                request.page,
                expectedFormat,
                request.startDate,
                request.endDate,
                request.plantId),
            Times.Once);
    }

    [Theory]
    [InlineData(1, "PLANT001", "1", "2024-01-01", "2024-12-31")]
    [InlineData(2, "PLANT002", "2", "2024-06-01", "2024-06-30")]
    [InlineData(3, "", "1", null, null)]
    public async Task GetKpiData_WithDifferentValidInputs_ReturnsOkResult(
        int affiliateId, string plantId, string page, string startDate, string endDate)
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
        var response = new GetKpiDataResponse { plantId = 1, kpiName = "Test" };

        _mockConfigServices
            .Setup(x => x.GetAffiliateCodeList(affiliateId))
            .Returns(affiliateCodeList);

        _mockCurrentServices
            .Setup(x => x.GetKpiDetailsAsync(
                It.IsAny<string>(),
                It.IsAny<string>(),
                It.IsAny<string>(),
                It.IsAny<string>(),
                It.IsAny<string>()))
            .ReturnsAsync(response);

        // Act
        var result = await _controller.GetKpiData(request);

        // Assert
        var okResult = Assert.IsType<OkObjectResult>(result);
        Assert.NotNull(okResult.Value);
    }

    [Fact]
    public async Task GetKpiData_WithNullAffiliateId_ReturnsUnauthorized()
    {
        // Arrange
        var request = new GetKpiDetailsRequest
        {
            affiliateId = null,
            plantId = "PLANT001",
            page = "1"
        };

        _mockConfigServices
            .Setup(x => x.GetAffiliateCodeList(null))
            .Returns((List<string>)null);

        // Act
        var result = await _controller.GetKpiData(request);

        // Assert
        Assert.IsType<UnauthorizedResult>(result);
    }

    [Fact]
    public async Task GetKpiData_VerifyAllParametersP
