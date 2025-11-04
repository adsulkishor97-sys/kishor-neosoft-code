using Xunit;
using Moq;
using AutoFixture;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

public class CurrentServicesTests
{
    private readonly Fixture _fixture;
    private readonly Mock<ICurrentRepository> _mockRepository;
    private readonly CurrentServices _service;

    public CurrentServicesTests()
    {
        _fixture = new Fixture();
        _mockRepository = new Mock<ICurrentRepository>();

        // Initialize service with mocked dependencies
        var mockConfiguration = new Mock<IConfiguration>();
        _service = new CurrentServices(_mockRepository.Object, mockConfiguration.Object);
    }

    [Fact]
    public async Task GetAffDistFinalOverallCriticalResult_ShouldReturnGroupedData_WhenValidInputsProvided()
    {
        // Arrange
        var overAllCriticalResult = _fixture.CreateMany<KpiNumeratorDenominatorAffiliateDistribution>(5).ToList();

        // Create matching affiliates for join operation
        var affiliates = overAllCriticalResult
            .Select(x => _fixture.Build<Affiliate>()
                                .With(a => a.affiliateCode, x.affiliates)
                                .With(a => a.affiliateName, _fixture.Create<string>())
                                .Create())
            .ToList();

        _mockRepository
            .Setup(repo => repo.GetAffiliateLists())
            .ReturnsAsync(affiliates);

        // Act
        var result = await _service.GetAffDistFinalOverallCriticalResult(overAllCriticalResult);

        // Assert
        Assert.NotNull(result);
        Assert.NotEmpty(result);
        Assert.All(result, item =>
        {
            Assert.False(string.IsNullOrEmpty(item.affiliateId));
            Assert.False(string.IsNullOrEmpty(item.name));
            Assert.True(item.overall >= 0);
            Assert.True(item.critical >= 0);
        });

        // Verify repository method called once
        _mockRepository.Verify(repo => repo.GetAffiliateLists(), Times.Once);
    }
}
