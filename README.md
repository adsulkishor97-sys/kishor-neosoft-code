using Xunit;
using Moq;
using System.Collections.Generic;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;

public class KpiControllerTests
{
    private readonly Mock<IConfigServices> _configServiceMock;
    private readonly Mock<ICurrentServices> _currentServiceMock;
    private readonly KpiController _controller;

    public KpiControllerTests()
    {
        _configServiceMock = new Mock<IConfigServices>();
        _currentServiceMock = new Mock<ICurrentServices>();
        _controller = new KpiController(_configServiceMock.Object, _currentServiceMock.Object);
    }

    [Fact]
    public async Task GetKpiData_ReturnsOk_WhenDataExists()
    {
        // Arrange
        var request = new GetKpiDetailsRequest
        {
            affiliateId = 1,
            plantId = "123",
            page = "Dashboard",
            startDate = "2025-01-01",
            endDate = "2025-02-01"
        };

        var affiliateCodes = new List<int> { 101, 102 };
        _configServiceMock.Setup(s => s.GetAffiliateCodeList(It.IsAny<int?>()))
            .Returns(affiliateCodes);

        var kpiResponse = new List<GetKpiDataResponse>
        {
            new GetKpiDataResponse { kpiCode = "KPI01", kpiName = "Efficiency" }
        };

        _currentServiceMock.Setup(s => s.GetKpiDetailsAsync(
            request.page!,
            It.IsAny<string>(),
            request.startDate!,
            request.endDate!,
            request.plantId!
        )).ReturnsAsync(kpiResponse);

        // Act
        var result = await _controller.GetKpiData(request);

        // Assert
        var okResult = Assert.IsType<OkObjectResult>(result);
        var data = Assert.IsAssignableFrom<List<GetKpiDataResponse>>(okResult.Value);
        Assert.Single(data);
        Assert.Equal("KPI01", data[0].kpiCode);
    }

    [Fact]
    public async Task GetKpiData_ReturnsNoContent_WhenNoData()
    {
        // Arrange
        var request = new GetKpiDetailsRequest
        {
            affiliateId = 1,
            plantId = "123",
            page = "Dashboard",
            startDate = "2025-01-01",
            endDate = "2025-02-01"
        };

        var affiliateCodes = new List<int> { 101 };
        _configServiceMock.Setup(s => s.GetAffiliateCodeList(It.IsAny<int?>()))
            .Returns(affiliateCodes);

        _currentServiceMock.Setup(s => s.GetKpiDetailsAsync(
            It.IsAny<string>(), It.IsAny<string>(), It.IsAny<string>(), It.IsAny<string>(), It.IsAny<string>()
        )).ReturnsAsync((List<GetKpiDataResponse>?)null);

        // Act
        var result = await _controller.GetKpiData(request);

        // Assert
        Assert.IsType<NoContentResult>(result);
    }

    [Fact]
    public async Task GetKpiData_ReturnsUnauthorized_WhenAffiliateListNull()
    {
        // Arrange
        var request = new GetKpiDetailsRequest
        {
            affiliateId = null
        };

        _configServiceMock.Setup(s => s.GetAffiliateCodeList(It.IsAny<int?>()))
            .Returns((List<int>)null!);

        // Act
        var result = await _controller.GetKpiData(request);

        // Assert
        Assert.IsType<UnauthorizedResult>(result);
    }
}
