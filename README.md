using Xunit;
using Moq;
using AutoFixture;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

public class CurrentServicesTests
{
    private readonly Fixture _fixture;
    private readonly Mock<ICurrentRepository> _currentRepoMock;
    private readonly CurrentServices _service;

    public CurrentServicesTests()
    {
        _fixture = new Fixture();
        _currentRepoMock = new Mock<ICurrentRepository>();
        var mockConfig = new Mock<IConfiguration>();
        _service = new CurrentServices(_currentRepoMock.Object, mockConfig.Object);
    }

    #region GetAffDistFinalOverallNumDenPlantResult

    [Fact]
    public async Task GetAffDistFinalOverallNumDenPlantResult_ShouldReturnGroupedData_WhenValidInput()
    {
        // Arrange
        var kpiId = _fixture.Create<int>();
        var plantId = _fixture.Create<int>();
        var overallNum = _fixture.Create<decimal>();
        var overallDen = _fixture.Create<decimal>();
        var plantName = _fixture.Create<string>();

        var overAllNum = new List<KpiNumeratorDenominatorAffiliateDistribution>
        {
            new KpiNumeratorDenominatorAffiliateDistribution
            {
                kpiid = kpiId,
                plantid = plantId,
                overallNumerator = overallNum
            }
        };

        var overAllDen = new List<KpiNumeratorDenominatorAffiliateDistribution>
        {
            new KpiNumeratorDenominatorAffiliateDistribution
            {
                kpiid = kpiId,
                plantid = plantId,
                overallDenominator = overallDen
            }
        };

        var plantList = new List<PlantList>
        {
            new PlantList
            {
                plantId = plantId,
                plantName = plantName
            }
        };

        _currentRepoMock.Setup(r => r.GetPlantLists()).ReturnsAsync(plantList);

        // Act
        var result = await _service.GetAffDistFinalOverallNumDenPlantResult(overAllNum, overAllDen);

        // Assert
        Assert.NotNull(result);
        Assert.NotEmpty(result);

        var first = result.First();
        Assert.Equal(plantId.ToString(), first.plantId);
        Assert.Equal(plantName, first.plantName);
        Assert.True(first.overall >= 0);
        Assert.True(first.critical >= 0);
        _currentRepoMock.Verify(r => r.GetPlantLists(), Times.Once);
    }

    [Fact]
    public async Task GetAffDistFinalOverallNumDenPlantResult_ShouldHandleZeroDenominator()
    {
        // Arrange
        var kpiId = _fixture.Create<int>();
        var plantId = _fixture.Create<int>();
        var overallNum = _fixture.Create<decimal>();
        var plantName = _fixture.Create<string>();

        var overAllNum = new List<KpiNumeratorDenominatorAffiliateDistribution>
        {
            new KpiNumeratorDenominatorAffiliateDistribution
            {
                kpiid = kpiId,
                plantid = plantId,
                overallNumerator = overallNum
            }
        };

        var overAllDen = new List<KpiNumeratorDenominatorAffiliateDistribution>
        {
            new KpiNumeratorDenominatorAffiliateDistribution
            {
                kpiid = kpiId,
                plantid = plantId,
                overallDenominator = 0 // zero denominator case
            }
        };

        var plantList = new List<PlantList>
        {
            new PlantList { plantId = plantId, plantName = plantName }
        };

        _currentRepoMock.Setup(r => r.GetPlantLists()).ReturnsAsync(plantList);

        // Act
        var result = await _service.GetAffDistFinalOverallNumDenPlantResult(overAllNum, overAllDen);

        // Assert
        Assert.NotNull(result);
        Assert.Single(result);
        Assert.Equal(overallNum, result.First().overall); // direct value since denom=0
        _currentRepoMock.Verify(r => r.GetPlantLists(), Times.Once);
    }

    [Fact]
    public async Task GetAffDistFinalOverallNumDenPlantResult_ShouldReturnEmpty_WhenNoMatchingPlants()
    {
        // Arrange
        var overAllNum = _fixture.CreateMany<KpiNumeratorDenominatorAffiliateDistribution>(2).ToList();
        var overAllDen = _fixture.CreateMany<KpiNumeratorDenominatorAffiliateDistribution>(2).ToList();

        _currentRepoMock.Setup(r => r.GetPlantLists()).ReturnsAsync(new List<PlantList>()); // no plants

        // Act
        var result = await _service.GetAffDistFinalOverallNumDenPlantResult(overAllNum, overAllDen);

        // Assert
        Assert.NotNull(result);
        Assert.Empty(result);
        _currentRepoMock.Verify(r => r.GetPlantLists(), Times.Once);
    }

    #endregion
}
