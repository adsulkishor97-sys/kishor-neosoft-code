using Xunit;
using Moq;
using AutoFixture;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.Extensions.Configuration;

public class CurrentServicesTests
{
    private readonly Fixture _fixture;
    private readonly Mock<ICurrentRepository> _mockRepository;
    private readonly CurrentServices _service;

    public CurrentServicesTests()
    {
        _fixture = new Fixture();
        _mockRepository = new Mock<ICurrentRepository>();
        var mockConfig = new Mock<IConfiguration>();
        _service = new CurrentServices(_mockRepository.Object, mockConfig.Object);
    }

    [Fact]
    public async Task GetAffDistFinalOverallCriticalResult_ShouldReturnExpectedGroupedData_WhenValidInput()
    {
        // Arrange
        var kpiDataList = _fixture.CreateMany<KpiNumeratorDenominatorAffiliateDistribution>(5).ToList();

        // Ensure all affiliates have matching codes for join
        var affiliateList = kpiDataList
            .Select(k => new Affiliate
            {
                affiliateCode = k.affiliates,
                affiliateName = _fixture.Create<string>()
            })
            .ToList();

        _mockRepository
            .Setup(x => x.GetAffiliateLists())
            .ReturnsAsync(affiliateList);

        // Act
        var result = await _service.GetAffDistFinalOverallCriticalResult(kpiDataList);

        // Assert
        Assert.NotNull(result);
        Assert.NotEmpty(result);

        foreach (var item in result)
        {
            Assert.False(string.IsNullOrWhiteSpace(item.affiliateId));
            Assert.False(string.IsNullOrWhiteSpace(item.name));
            Assert.InRange(item.overall, decimal.MinValue, decimal.MaxValue);
            Assert.InRange(item.critical, decimal.MinValue, decimal.MaxValue);
        }

        // Verify repository was called exactly once
        _mockRepository.Verify(x => x.GetAffiliateLists(), Times.Once);
    }

    [Fact]
    public async Task GetAffDistFinalOverallCriticalResult_ShouldReturnEmptyList_WhenNoAffiliatesMatch()
    {
        // Arrange
        var kpiDataList = _fixture.CreateMany<KpiNumeratorDenominatorAffiliateDistribution>(5).ToList();
        var emptyAffiliateList = new List<Affiliate>(); // no matches

        _mockRepository
            .Setup(x => x.GetAffiliateLists())
            .ReturnsAsync(emptyAffiliateList);

        // Act
        var result = await _service.GetAffDistFinalOverallCriticalResult(kpiDataList);

        // Assert
        Assert.NotNull(result);
        Assert.Empty(result);

        _mockRepository.Verify(x => x.GetAffiliateLists(), Times.Once);
    }
}
    #region GetAffDistFinalOverallCriticalResult
    [Theory]
    [ClassData(typeof(GetAffDistFinalOverallCriticalResultGenerator))]
    public async Task GetAffDistFinalOverallCriticalResult_ShouldReturnData_WhenRepositoryReturnsData(List<KpiNumeratorDenominatorAffiliateDistribution> request)
    {
        // Arrange
        var dbResponse = Enumerable.Range(0, 3).Select(index => Mock.Of<AffiliateList>())

.ToList();

        _currRepositoryMock.Setup(repo => repo.GetAffiliateLists()).ReturnsAsync(dbResponse);


        //Act
        var result = await _currServices.GetAffDistFinalOverallCriticalResult(request);
        // Assert
        Assert.NotNull(result);
        Assert.IsType<List<GroupedData>>(result);
    }

    #endregion
