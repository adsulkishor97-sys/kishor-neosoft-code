public class GetAssetDistributionRequest
{
    public string? page { get; set; }
    public string? startDate { get; set; }
    public string? endDate { get; set; }
    public string? kpiCode { get; set; }
    public string? subCategory { get; set; }
    public int? affiliateId { get; set; }
    public int? plantId { get; set; }
    public string category { get; set; } = string.Empty;
}

public class AssetGroupedData
{
    public string? affiliateId { get; set; }
    public string? name { get; set; }
    public decimal overall { get; set; }
    public decimal critical { get; set; }
    public string? plantId { get; set; }
    public string? plantName { get; set; }
}

public class GetCaseHierarchyRequest
{
    public string? bigDataAffiliateIdList { get; set; }
}

public interface IConfigServices
{
    List<int> GetAffiliateCodeList(int? affiliateId);
}

public interface ICurrentServices
{
    Task<List<AssetGroupedData>?> GetAssetDistributionAsync(GetAssetDistributionRequest request, string affiliateRequest);
}
🧪 Unit Tests (xUnit + Moq)
csharp
Copy code
using Xunit;
using Moq;
using Microsoft.AspNetCore.Mvc;
using System.Collections.Generic;
using System.Threading.Tasks;

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

    // ✅ Test 1: Should return Ok with data
    [Fact]
    public async Task GetAssetDistribution_ReturnsOk_WhenDataExists()
    {
        // Arrange
        var request = new GetAssetDistributionRequest
        {
            affiliateId = 1,
            page = "global"
        };

        var affiliateCodes = new List<int> { 10, 20 };
        var expectedResult = new List<AssetGroupedData>
        {
            new AssetGroupedData { affiliateId = "10", name = "Plant A", overall = 100 },
            new AssetGroupedData { affiliateId = "20", name = "Plant B", overall = 200 }
        };

        _configServiceMock.Setup(x => x.GetAffiliateCodeList(request.affiliateId)).Returns(affiliateCodes);

        _currentServiceMock
            .Setup(x => x.GetAssetDistributionAsync(It.IsAny<GetAssetDistributionRequest>(), It.IsAny<string>()))
            .ReturnsAsync(expectedResult);

        // Act
        var result = await _controller.GetAssetDistribution(request);

        // Assert
        var okResult = Assert.IsType<OkObjectResult>(result);
        var response = Assert.IsAssignableFrom<List<AssetGroupedData>>(okResult.Value);
        Assert.Equal(2, response.Count);
        Assert.Equal("Plant A", response[0].name);
    }

    // 🚫 Test 2: Should return NoContent when data is null
    [Fact]
    public async Task GetAssetDistribution_ReturnsNoContent_WhenDataIsNull()
    {
        // Arrange
        var request = new GetAssetDistributionRequest { affiliateId = 2 };

        _configServiceMock.Setup(x => x.GetAffiliateCodeList(request.affiliateId)).Returns(new List<int> { 10 });

        _currentServiceMock
            .Setup(x => x.GetAssetDistributionAsync(It.IsAny<GetAssetDistributionRequest>(), It.IsAny<string>()))
            .ReturnsAsync((List<AssetGroupedData>?)null);

        // Act
        var result = await _controller.GetAssetDistribution(request);

        // Assert
        Assert.IsType<NoContentResult>(result);
    }

    // 🔒 Test 3: Should return Unauthorized if affiliate list is null
    [Fact]
    public async Task GetAssetDistribution_ReturnsUnauthorized_WhenAffiliateListIsNull()
    {
        // Arrange
        var request = new GetAssetDistributionRequest { affiliateId = 3 };

        _configServiceMock.Setup(x => x.GetAffiliateCodeList(request.affiliateId)).Returns(new List<int>());

        // Act
        var result = await _controller.GetAssetDistribution(request);

        // Assert
        var unauthorizedResult = Assert.IsType<UnauthorizedResult>(result);
        Assert.Equal(401, unauthorizedResult.StatusCode);
    }

    // ⚠️ Optional: Exception handling test
    [Fact]
    public async Task GetAssetDistribution_ThrowsException_ShouldPropagate()
    {
        // Arrange
        var request = new GetAssetDistributionRequest { affiliateId = 4 };

        _configServiceMock.Setup(x => x.GetAffiliateCodeList(request.affiliateId))
            .Throws(new System.Exception("DB connection failed"));

        // Act & Assert
        await Assert.ThrowsAsync<System.Exception>(() => _controller.GetAssetDistribution(request));
    }
