using Xunit;
using Moq;
using AutoFixture;
using System.Collections.Generic;
using System.Linq;
using System.Reflection;
using System.Threading.Tasks;

public class AssetService_HandleAvailabilityKpi_Tests
{
    private readonly IFixture _fixture;
    private readonly Mock<ICurrentRepository> _mockRepo;
    private readonly Mock<IAssetHelper> _mockHelper;
    private readonly AssetService _service;

    public AssetService_HandleAvailabilityKpi_Tests()
    {
        _fixture = new Fixture();
        _mockRepo = new Mock<ICurrentRepository>();
        _mockHelper = new Mock<IAssetHelper>();

        _service = new AssetService(_mockRepo.Object, _mockHelper.Object);
    }

    [Fact]
    public async Task HandleAvailabilityKpi_ShouldReturnProcessedData_WhenRepositoryReturnsValidData()
    {
        // Arrange
        var item = _fixture.Build<AffiliateDistribution>()
                           .With(x => x.overalN, _fixture.Create<string>())
                           .Create();

        var affiliateRequest1 = _fixture.Create<string>();
        var request = _fixture.Create<GetAffiliatePlantDistributionRequest>();
        var startDate = _fixture.Create<string>();
        var endDate = _fixture.Create<string>();
        var quotedPmCodes = _fixture.Create<string>();

        var fakeData = _fixture.CreateMany<KpiNumeratorDenominatorAffiliateDistribution>(3).ToList();

        _mockRepo
            .Setup(r => r.ExecuteBigDataQuery_New<KpiNumeratorDenominatorAffiliateDistribution>(
                It.IsAny<string>(),
                It.IsAny<Func<DbDataReader, KpiNumeratorDenominatorAffiliateDistribution>>()))
            .ReturnsAsync(fakeData);

        _mockHelper
            .Setup(h => h.GetAffDistFinalOverallCriticalResult(It.IsAny<List<KpiNumeratorDenominatorAffiliateDistribution>>()))
            .ReturnsAsync(_fixture.CreateMany<GroupedData>(2).ToList());

        // Use reflection to access the private async method
        var method = typeof(AssetService).GetMethod("HandleAvailabilityKpi", BindingFlags.NonPublic | BindingFlags.Instance);

        // Act
        var task = (Task<List<GroupedData>>)method!.Invoke(_service, new object[]
        {
            item, affiliateRequest1, request, startDate, endDate, quotedPmCodes
        })!;
        var result = await task;

        // Assert
        Assert.NotNull(result);
        Assert.NotEmpty(result);
        _mockRepo.Verify(r => r.ExecuteBigDataQuery_New<KpiNumeratorDenominatorAffiliateDistribution>(
            It.IsAny<string>(),
            It.IsAny<Func<DbDataReader, KpiNumeratorDenominatorAffiliateDistribution>>()), Times.Once);
    }

    [Fact]
    public async Task HandleAvailabilityKpi_ShouldReturnEmptyList_WhenRepositoryReturnsNull()
    {
        // Arrange
        var item = _fixture.Build<AffiliateDistribution>()
                           .With(x => x.overalN, _fixture.Create<string>())
                           .Create();

        _mockRepo
            .Setup(r => r.ExecuteBigDataQuery_New<KpiNumeratorDenominatorAffiliateDistribution>(
                It.IsAny<string>(),
                It.IsAny<Func<DbDataReader, KpiNumeratorDenominatorAffiliateDistribution>>()))
            .ReturnsAsync((List<KpiNumeratorDenominatorAffiliateDistribution>?)null);

        _mockHelper
            .Setup(h => h.GetAffDistFinalOverallCriticalResult(It.IsAny<List<KpiNumeratorDenominatorAffiliateDistribution>>()))
            .ReturnsAsync(new List<GroupedData>());

        var method = typeof(AssetService).GetMethod("HandleAvailabilityKpi", BindingFlags.NonPublic | BindingFlags.Instance);

        // Act
        var task = (Task<List<GroupedData>>)method!.Invoke(_service, new object[]
        {
            _fixture.Create<AffiliateDistribution>(),
            _fixture.Create<string>(),
            _fixture.Create<GetAffiliatePlantDistributionRequest>(),
            _fixture.Create<string>(),
            _fixture.Create<string>(),
            _fixture.Create<string>()
        })!;
        var result = await task;

        // Assert
        Assert.NotNull(result);
        Assert.Empty(result);

        _mockRepo.Verify(r => r.ExecuteBigDataQuery_New<KpiNumeratorDenominatorAffiliateDistribution>(
            It.IsAny<string>(),
            It.IsAny<Func<DbDataReader, KpiNumeratorDenominatorAffiliateDistribution>>()), Times.Once);
    }
}
