using Xunit;
using Moq;
using AutoFixture;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

public class CurrentServicesTests
{
    private readonly Fixture _fixture;
    private readonly Mock<ICurrentRepository> _currRepositoryMock;
    private readonly CurrentServices _currServices;

    public CurrentServicesTests()
    {
        _fixture = new Fixture();
        _currRepositoryMock = new Mock<ICurrentRepository>();
        var mockConfig = new Mock<IConfiguration>();
        _currServices = new CurrentServices(_currRepositoryMock.Object, mockConfig.Object);
    }

    #region GetAffDistFinalOverallNumAssetResult Tests

    [Theory]
    [InlineData("U")]
    [InlineData("AC")]
    [InlineData("Other")]
    public async Task GetAffDistFinalOverallNumAssetResult_ShouldReturnExpectedGroupedData_ForAllCategories(string category)
    {
        // Arrange
        var kpiId = _fixture.Create<int>();
        var plantId = _fixture.Create<int>();
        var plantName = _fixture.Create<string>();
        var overallNumerator = _fixture.Create<decimal>();
        var sapId = _fixture.Create<string>();
        var template = _fixture.Create<string>();
        var tidnr = _fixture.Create<string>();
        var pltxt = _fixture.Create<string>();

        var overAllNum = new List<KpiNumeratorDenominatorAffiliateDistribution>
        {
            new KpiNumeratorDenominatorAffiliateDistribution
            {
                kpiid = kpiId,
                plantid = plantId,
                overallNumerator = overallNumerator,
                sap_id = sapId,
                template = template,
                tidnr = tidnr,
                pltxt = pltxt
            },
            // duplicate for grouping behavior
            new KpiNumeratorDenominatorAffiliateDistribution
            {
                kpiid = kpiId,
                plantid = plantId,
                overallNumerator = overallNumerator,
                sap_id = sapId,
                template = template,
                tidnr = tidnr,
                pltxt = pltxt
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

        _currRepositoryMock.Setup(r => r.GetPlantLists()).ReturnsAsync(plantList);

        // Act
        var result = await _currServices.GetAffDistFinalOverallNumAssetResult(overAllNum, category);

        // Assert
        Assert.NotNull(result);
        Assert.NotEmpty(result);

        var first = result.First();

        Assert.Equal(plantId.ToString(), first.plantId);
        Assert.Equal(sapId, first.sapId);
        Assert.Equal(template, first.template);
        Assert.Equal(tidnr, first.tidnr);
        Assert.Equal(pltxt, first.pltxt);

        // grouping should sum the numerator
        Assert.Equal(overallNumerator * 2, first.overall);

        _currRepositoryMock.Verify(r => r.GetPlantLists(), Times.Once);
    }

    [Fact]
    public async Task GetAffDistFinalOverallNumAssetResult_ShouldReturnEmpty_WhenNoPlantMatch()
    {
        // Arrange
        var overAllNum = _fixture.CreateMany<KpiNumeratorDenominatorAffiliateDistribution>(2).ToList();

        _currRepositoryMock.Setup(r => r.GetPlantLists())
            .ReturnsAsync(new List<PlantList>()); // no matching plant

        // Act
        var result = await _currServices.GetAffDistFinalOverallNumAssetResult(overAllNum, "U");

        // Assert
        Assert.NotNull(result);
        Assert.Empty(result);
        _currRepositoryMock.Verify(r => r.GetPlantLists(), Times.Once);
    }

    [Fact]
    public async Task GetAffDistFinalOverallNumAssetResult_ShouldHandleNullTemplateTidnrPltxtGracefully()
    {
        // Arrange
        var kpiId = _fixture.Create<int>();
        var plantId = _fixture.Create<int>();
        var overallNumerator = _fixture.Create<decimal>();
        var plantName = _fixture.Create<string>();

        var overAllNum = new List<KpiNumeratorDenominatorAffiliateDistribution>
        {
            new KpiNumeratorDenominatorAffiliateDistribution
            {
                kpiid = kpiId,
                plantid = plantId,
                overallNumerator = overallNumerator,
                sap_id = null,
                template = null,
                tidnr = null,
                pltxt = null
            }
        };

        var plantList = new List<PlantList>
        {
            new PlantList { plantId = plantId, plantName = plantName }
        };

        _currRepositoryMock.Setup(r => r.GetPlantLists()).ReturnsAsync(plantList);

        // Act
        var result = await _currServices.GetAffDistFinalOverallNumAssetResult(overAllNum, "AC");

        // Assert
        Assert.NotNull(result);
        Assert.Single(result);
        Assert.Null(result.First().sapId);
        Assert.Null(result.First().template);
        Assert.Null(result.First().tidnr);
        Assert.Null(result.First().pltxt);
        _currRepositoryMock.Verify(r => r.GetPlantLists(), Times.Once);
    }

    #endregion
}
