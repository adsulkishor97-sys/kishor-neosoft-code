using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using AutoFixture;
using Moq;
using Xunit;

public class AssetServiceTests
{
    private readonly Mock<ICurrentRepository> _currentRepositoryMock;
    private readonly Fixture _fixture;
    private readonly AssetService _service;

    public AssetServiceTests()
    {
        _fixture = new Fixture();
        _currentRepositoryMock = new Mock<ICurrentRepository>();
        _service = new AssetService(_currentRepositoryMock.Object);
    }

    [Theory]
    [InlineData("U")]
    [InlineData("AC")]
    [InlineData("Other")]
    public async Task GetAffDistFinalOverallNumAssetResult_ShouldReturnGroupedData_BasedOnCategory(string category)
    {
        // Arrange
        var overAllNumList = _fixture.CreateMany<KpiNumeratorDenominatorAffiliateDistribution>(3).ToList();
        var plantList = _fixture.CreateMany<PlantList>(3).ToList();

        // Make sure Plant IDs match between both lists for join
        for (int i = 0; i < overAllNumList.Count; i++)
        {
            overAllNumList[i].plantid = plantList[i].plantId;
            overAllNumList[i].kpiid = i + 1;
            overAllNumList[i].overallNumerator = _fixture.Create<decimal>();
        }

        _currentRepositoryMock
            .Setup(x => x.GetPlantLists())
            .ReturnsAsync(plantList);

        // Act
        var result = await _service.GetAffDistFinalOverallNumAssetResult(overAllNumList, category);

        // Assert
        Assert.NotNull(result);
        Assert.NotEmpty(result);
        Assert.All(result, r =>
        {
            Assert.NotNull(r.plantId);
            Assert.True(r.overall >= 0);
        });

        _currentRepositoryMock.Verify(x => x.GetPlantLists(), Times.Once);
    }
}
