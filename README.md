using Xunit;
using Moq;
using AutoFixture;
using System;
using System.Linq;
using System.Reflection;
using System.Threading.Tasks;
using System.Collections.Generic;

public class AssetServicePrivateTests
{
    private readonly IFixture _fixture;
    private readonly Mock<ICurrentRepository> _mockRepo;
    private readonly AssetService _service;

    public AssetServicePrivateTests()
    {
        _fixture = new Fixture();
        _mockRepo = new Mock<ICurrentRepository>();
        _service = new AssetService(_mockRepo.Object);
    }

    [Fact]
    public async Task GetPmCodesAsync_ShouldReturnQuotedPmCodes_WhenPmCodeExists()
    {
        // Arrange
        var request = _fixture.Create<KpiInputRequest>();

        var assetClassMatch = _fixture.Create<string>();
        request.subcategory = assetClassMatch;

        // random pmCodes separated by commas
        var pmCodes = string.Join(",", _fixture.CreateMany<string>(3));

        var fakeData = _fixture.Build<AssetClassResponse>()
                               .With(x => x.assetClass, assetClassMatch)
                               .With(x => x.pmCode, pmCodes)
                               .CreateMany(5)
                               .ToList();

        _mockRepo.Setup(r => r.GetAssetClassAsync()).ReturnsAsync(fakeData);

        // Use reflection to access private method
        var method = typeof(AssetService).GetMethod("GetPmCodesAsync", BindingFlags.NonPublic | BindingFlags.Instance);

        // Act
        var task = (Task<string>)method!.Invoke(_service, new object[] { request })!;
        var result = await task;

        // Assert
        Assert.NotNull(result);
        Assert.All(pmCodes.Split(','), code =>
        {
            Assert.Contains($"'{code.Trim()}'", result);
        });

        _mockRepo.Verify(r => r.GetAssetClassAsync(), Times.Once);
    }

    [Fact]
    public async Task GetPmCodesAsync_ShouldReturnEmpty_WhenPmCodeIsMissing()
    {
        // Arrange
        var request = _fixture.Create<KpiInputRequest>();
        request.subcategory = _fixture.Create<string>();

        var fakeData = _fixture.Build<AssetClassResponse>()
                               .With(x => x.assetClass, _fixture.Create<string>())
                               .With(x => x.pmCode, (string?)null)
                               .CreateMany(3)
                               .ToList();

        _mockRepo.Setup(r => r.GetAssetClassAsync()).ReturnsAsync(fakeData);

        // Reflection to access private async method
        var method = typeof(AssetService).GetMethod("GetPmCodesAsync", BindingFlags.NonPublic | BindingFlags.Instance);

        // Act
        var task = (Task<string>)method!.Invoke(_service, new object[] { request })!;
        var result = await task;

        // Assert
        Assert.NotNull(result);
        Assert.Equal(string.Empty, result);

        _mockRepo.Verify(r => r.GetAssetClassAsync(), Times.Once);
    }
}
