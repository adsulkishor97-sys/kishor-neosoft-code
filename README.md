using Xunit;
using Moq;
using AutoFixture;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

// ✅ TEST CLASS
public class AssetServiceTests
{
    private readonly IFixture _fixture;
    private readonly Mock<IAssetRepository> _mockRepo;
    private readonly AssetService _service;

    public AssetServiceTests()
    {
        _fixture = new Fixture();
        _mockRepo = new Mock<IAssetRepository>();
        _service = new AssetService(_mockRepo.Object);
    }

    [Fact]
    public async Task GetAssetDesignSubgroupBySapIds_ShouldGroupAndMapCorrectly_UsingAutoFixture()
    {
        // Arrange
        var request = _fixture.Create<DesignDataBySapIdRequest>();

        // Generate random category IDs/names
        var category1 = _fixture.Create<string>();
        var category2 = _fixture.Create<string>();

        // Generate multiple random entries with same and different categories
        var fakeData = new List<AssetDesignSubgroupBySapIdRaw>
        {
            _fixture.Build<AssetDesignSubgroupBySapIdRaw>()
                .With(x => x.categoryId, category1)
                .With(x => x.categoryName, _fixture.Create<string>())
                .Create(),
            _fixture.Build<AssetDesignSubgroupBySapIdRaw>()
                .With(x => x.categoryId, category1)
                .With(x => x.categoryName, _fixture.Create<string>())
                .Create(),
            _fixture.Build<AssetDesignSubgroupBySapIdRaw>()
                .With(x => x.categoryId, category2)
                .With(x => x.categoryName, _fixture.Create<string>())
                .Create()
        };

        _mockRepo
            .Setup(r => r.GetAssetDesignSubgroupBySapIds(It.IsAny<DesignDataBySapIdRequest>()))
            .ReturnsAsync(fakeData);

        // Act
        var result = await _service.GetAssetDesignSubgroupBySapIds(request);

        // Assert
        Assert.NotNull(result);
        Assert.True(result.Count >= 2); // should have at least two grouped categories

        // Each group should have corresponding subgroup data
        foreach (var group in result)
        {
            Assert.NotNull(group.categoryId);
            Assert.NotNull(group.categoryName);
            Assert.NotNull(group.assetDesignSubgroupDataItems);
            Assert.All(group.assetDesignSubgroupDataItems, item =>
            {
                Assert.False(string.IsNullOrWhiteSpace(item.mark));
                Assert.False(string.IsNullOrWhiteSpace(item.size));
                Assert.False(string.IsNullOrWhiteSpace(item.rating));
            });
        }

        // Verify repository interaction
        _mockRepo.Verify(r => r.GetAssetDesignSubgroupBySapIds(request), Times.Once);
    }

    [Fact]
    public async Task GetAssetDesignSubgroupBySapIds_ShouldReturnEmptyList_WhenRepositoryReturnsEmpty()
    {
        // Arrange
        var request = _fixture.Create<DesignDataBySapIdRequest>();
        _mockRepo
            .Setup(r => r.GetAssetDesignSubgroupBySapIds(It.IsAny<DesignDataBySapIdRequest>()))
            .ReturnsAsync(new List<AssetDesignSubgroupBySapIdRaw>());

        // Act
        var result = await _service.GetAssetDesignSubgroupBySapIds(request);

        // Assert
        Assert.NotNull(result);
        Assert.Empty(result);

        _mockRepo.Verify(r => r.GetAssetDesignSubgroupBySapIds(request), Times.Once);
    }
}
