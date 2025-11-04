using Xunit;
using Moq;
using AutoFixture;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using System;
using Microsoft.Extensions.Configuration;

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

    #region GetAffDistFinalNumeratorDenominatorResult Tests

    [Fact]
    public async Task GetAffDistFinalNumeratorDenominatorResult_ShouldReturnExpectedGroupedData_WhenRepositoryReturnsData()
    {
        // Arrange
        var kpiId = _fixture.Create<int>();
        var affiliateCode = _fixture.Create<int>();
        var affiliateName = _fixture.Create<string>();

        var overallNum = _fixture.Build<KpiNumeratorDenominatorAffiliateDistribution>()
            .With(x => x.kpiid, kpiId)
            .With(x => x.affiliates, affiliateCode)
            .With(x => x.overallNumerator, _fixture.Create<decimal>())
            .CreateMany(2).ToList();

        var overallDen = overallNum.Select(n => _fixture.Build<KpiNumeratorDenominatorAffiliateDistribution>()
            .With(x => x.kpiid, n.kpiid)
            .With(x => x.affiliates, n.affiliates)
            .With(x => x.overallDenominator, _fixture.Create<decimal>() + 1) // avoid divide by zero
            .Create()).ToList();

        var criticalNum = overallNum.Select(n => _fixture.Build<KpiNumeratorDenominatorAffiliateDistribution>()
            .With(x => x.kpiid, n.kpiid)
            .With(x => x.affiliates, n.affiliates)
            .With(x => x.criticalNumerator, _fixture.Create<decimal>())
            .Create()).ToList();

        var criticalDen = overallNum.Select(n => _fixture.Build<KpiNumeratorDenominatorAffiliateDistribution>()
            .With(x => x.kpiid, n.kpiid)
            .With(x => x.affiliates, n.affiliates)
            .With(x => x.criticalDenominator, _fixture.Create<decimal>() + 1)
            .Create()).ToList();

        var affiliates = _fixture.Build<AffiliateList>()
            .With(a => a.affiliateCode, affiliateCode)
            .With(a => a.affiliateName, affiliateName)
            .CreateMany(3).ToList();

        _currRepositoryMock.Setup(r => r.GetAffiliateLists()).ReturnsAsync(affiliates);

        // Act
        var result = await _currServices.GetAffDistFinalNumeratorDenominatorResult(
            overallNum, overallDen, criticalNum, criticalDen);

        // Assert
        Assert.NotNull(result);
        Assert.All(result, r =>
        {
            Assert.NotNull(r.affiliateId);
            Assert.NotNull(r.name);
            Assert.True(r.overall >= 0);
            Assert.True(r.critical >= 0);
        });

        _currRepositoryMock.Verify(r => r.GetAffiliateLists(), Times.Once);
    }

    [Fact]
    public async Task GetAffDistFinalNumeratorDenominatorResult_ShouldHandleZeroDenominatorsGracefully()
    {
        // Arrange
        var kpiId = _fixture.Create<int>();
        var affiliateCode = _fixture.Create<int>();
        var affiliateName = _fixture.Create<string>();

        var numeratorValue = _fixture.Create<decimal>();
        var criticalValue = _fixture.Create<decimal>();

        var overallNum = _fixture.Build<KpiNumeratorDenominatorAffiliateDistribution>()
            .With(x => x.kpiid, kpiId)
            .With(x => x.affiliates, affiliateCode)
            .With(x => x.overallNumerator, numeratorValue)
            .CreateMany(1).ToList();

        var overallDen = _fixture.Build<KpiNumeratorDenominatorAffiliateDistribution>()
            .With(x => x.kpiid, kpiId)
            .With(x => x.affiliates, affiliateCode)
            .With(x => x.overallDenominator, 0) // zero denominator
            .CreateMany(1).ToList();

        var criticalNum = _fixture.Build<KpiNumeratorDenominatorAffiliateDistribution>()
            .With(x => x.kpiid, kpiId)
            .With(x => x.affiliates, affiliateCode)
            .With(x => x.criticalNumerator, criticalValue)
            .CreateMany(1).ToList();

        var criticalDen = _fixture.Build<KpiNumeratorDenominatorAffiliateDistribution>()
            .With(x => x.kpiid, kpiId)
            .With(x => x.affiliates, affiliateCode)
            .With(x => x.criticalDenominator, 0)
            .CreateMany(1).ToList();

        var affiliates = _fixture.Build<AffiliateList>()
            .With(a => a.affiliateCode, affiliateCode)
            .With(a => a.affiliateName, affiliateName)
            .CreateMany(2).ToList();

        _currRepositoryMock.Setup(r => r.GetAffiliateLists()).ReturnsAsync(affiliates);

        // Act
        var result = await _currServices.GetAffDistFinalNumeratorDenominatorResult(
            overallNum, overallDen, criticalNum, criticalDen);

        // Assert
        Assert.NotNull(result);
        Assert.Single(result);
        var item = result.First();

        // When denominator is 0, overall == numerator
        Assert.Equal(numeratorValue, item.overall);
        Assert.Equal(criticalValue, item.critical);
    }

    [Fact]
    public async Task GetAffDistFinalNumeratorDenominatorResult_ShouldReturnEmptyList_WhenNoMatchingAffiliateFound()
    {
        // Arrange
        var overAllNum = _fixture.CreateMany<KpiNumeratorDenominatorAffiliateDistribution>(2).ToList();
        var overAllDen = _fixture.CreateMany<KpiNumeratorDenominatorAffiliateDistribution>(2).ToList();
        var criticalNum = _fixture.CreateMany<KpiNumeratorDenominatorAffiliateDistribution>(2).ToList();
        var criticalDen = _fixture.CreateMany<KpiNumeratorDenominatorAffiliateDistribution>(2).ToList();

        var affiliates = new List<AffiliateList>(); // no matches

        _currRepositoryMock.Setup(r => r.GetAffiliateLists()).ReturnsAsync(affiliates);

        // Act
        var result = await _currServices.GetAffDistFinalNumeratorDenominatorResult(
            overAllNum, overAllDen, criticalNum, criticalDen);

        // Assert
        Assert.NotNull(result);
        Assert.Empty(result);
        _currRepositoryMock.Verify(r => r.GetAffiliateLists(), Times.Once);
    }

    #endregion
}
