#region GetAffDistFinalOverallCriticalResult

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

    [Theory]
    [ClassData(typeof(GetAffDistFinalOverallCriticalResultGenerator))]
    public async Task GetAffDistFinalOverallCriticalResult_ShouldReturnExpectedGroupedData_WhenValidInput(
        List<KpiNumeratorDenominatorAffiliateDistribution> request)
    {
        // Arrange
        var affiliates = request
            .Select(r => new AffiliateList
            {
                affiliateCode = r.affiliates,
                affiliateName = _fixture.Create<string>()
            })
            .ToList();

        _currRepositoryMock
            .Setup(repo => repo.GetAffiliateLists())
            .ReturnsAsync(affiliates);

        // Act
        var result = await _currServices.GetAffDistFinalOverallCriticalResult(request);

        // Assert
        Assert.NotNull(result);
        Assert.NotEmpty(result);
        Assert.All(result, item =>
        {
            Assert.False(string.IsNullOrEmpty(item.affiliateId));
            Assert.False(string.IsNullOrEmpty(item.name));
            Assert.InRange(item.overall, decimal.MinValue, decimal.MaxValue);
            Assert.InRange(item.critical, decimal.MinValue, decimal.MaxValue);
        });

        _currRepositoryMock.Verify(repo => repo.GetAffiliateLists(), Times.Once);
    }

    [Fact]
    public async Task GetAffDistFinalOverallCriticalResult_ShouldReturnEmptyList_WhenNoAffiliatesMatch()
    {
        // Arrange
        var request = _fixture.CreateMany<KpiNumeratorDenominatorAffiliateDistribution>(5).ToList();

        _currRepositoryMock
            .Setup(repo => repo.GetAffiliateLists())
            .ReturnsAsync(new List<AffiliateList>()); // No affiliates

        // Act
        var result = await _currServices.GetAffDistFinalOverallCriticalResult(request);

        // Assert
        Assert.NotNull(result);
        Assert.Empty(result);
        _currRepositoryMock.Verify(repo => repo.GetAffiliateLists(), Times.Once);
    }
}

#endregion

#region Data Generator

public class GetAffDistFinalOverallCriticalResultGenerator : TheoryData<List<KpiNumeratorDenominatorAffiliateDistribution>>
{
    public GetAffDistFinalOverallCriticalResultGenerator()
    {
        var fixture = new Fixture();
        var requestData = fixture.CreateMany<KpiNumeratorDenominatorAffiliateDistribution>(5).ToList();

        // Ensure some duplicate affiliate IDs for grouping logic coverage
        requestData[0].affiliates = requestData[1].affiliates;

        Add(requestData);
    }
}

#endregion
