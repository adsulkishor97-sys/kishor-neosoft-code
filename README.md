using Xunit;
using Moq;
using AutoFixture;
using System.Collections.Generic;
using System.Threading.Tasks;
using System.Linq;
using System;
using YourNamespace.Models;
using YourNamespace.Services;

public class PerformanceSummaryPlantServiceTests
{
    private readonly IFixture _fixture;
    private readonly Mock<IConfigRepository> _mockConfigRepository;
    private readonly Mock<ICurrentRepository> _mockCurrentRepository;
    private readonly PerformanceSummaryService _service; // Replace with actual service name

    public PerformanceSummaryPlantServiceTests()
    {
        _fixture = new Fixture();
        _mockConfigRepository = new Mock<IConfigRepository>();
        _mockCurrentRepository = new Mock<ICurrentRepository>();

        _service = new PerformanceSummaryService(
            _mockConfigRepository.Object,
            _mockCurrentRepository.Object
        );
    }

    // ✅ Test 1: Returns OK result when dependencies return data
    [Fact]
    public async Task PerformanceSummaryPlantAsync_ReturnsValidResponse_WhenDependenciesSucceed()
    {
        // Arrange
        var request = _fixture.Create<GetKpiPerformanceRequest>();
        var affiliateRequest = _fixture.Create<string>();

        var fakePlantDetails = _fixture.CreateMany<GetCaseHierarchyResponse>(3).ToList();
        fakePlantDetails.First().plantName = _fixture.Create<string>();

        var fakeAffiliates = _fixture.CreateMany<Affiliate>(3).ToList();

        _mockConfigRepository
            .Setup(repo => repo.GetCaseHierarchyAsync(It.IsAny<GetCaseHierarchyRequest>()))
            .ReturnsAsync(fakePlantDetails);

        _mockCurrentRepository
            .Setup(repo => repo.GetAffiliateLists())
            .ReturnsAsync(fakeAffiliates);

        // Act
        var result = await _service.PerformanceSummaryPlantAsync(request, affiliateRequest);

        // Assert
        Assert.NotNull(result);
        Assert.IsType<KpiPerformanceResponse>(result);
        Assert.Equal(request.performanceSummary, result.performanceSummary);

        _mockConfigRepository.Verify(r => r.GetCaseHierarchyAsync(It.IsAny<GetCaseHierarchyRequest>()), Times.Once);
        _mockCurrentRepository.Verify(r => r.GetAffiliateLists(), Times.AtLeastOnce);
    }

    // ⚠️ Test 2: Returns default response when exception occurs
    [Fact]
    public async Task PerformanceSummaryPlantAsync_ReturnsEmptyResponse_WhenExceptionThrown()
    {
        // Arrange
        var request = _fixture.Create<GetKpiPerformanceRequest>();
        var affiliateRequest = _fixture.Create<string>();

        _mockConfigRepository
            .Setup(repo => repo.GetCaseHierarchyAsync(It.IsAny<GetCaseHierarchyRequest>()))
            .ThrowsAsync(new Exception(_fixture.Create<string>()));

        // Act
        var result = await _service.PerformanceSummaryPlantAsync(request, affiliateRequest);

        // Assert
        Assert.NotNull(result);
        Assert.Null(result.performanceSummary); // default due to catch block
    }

    // ✅ Test 3: Verifies repositories called with expected data
    [Fact]
    public async Task PerformanceSummaryPlantAsync_CallsRepositoriesWithExpectedInput()
    {
        // Arrange
        var request = _fixture.Create<GetKpiPerformanceRequest>();
        var affiliateRequest = _fixture.Create<string>();

        _mockConfigRepository
            .Setup(repo => repo.GetCaseHierarchyAsync(It.IsAny<GetCaseHierarchyRequest>()))
            .ReturnsAsync(_fixture.CreateMany<GetCaseHierarchyResponse>(2).ToList());

        _mockCurrentRepository
            .Setup(repo => repo.GetAffiliateLists())
            .ReturnsAsync(_fixture.CreateMany<Affiliate>(2).ToList());

        // Act
        var result = await _service.PerformanceSummaryPlantAsync(request, affiliateRequest);

        // Assert
        _mockConfigRepository.Verify(r => r.GetCaseHierarchyAsync(
            It.Is<GetCaseHierarchyRequest>(x => x.affiliateIdList == request.affiliateId.ToString())
        ), Times.Once);

        Assert.NotNull(result);
    }
}
