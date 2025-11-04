using Xunit;
using Moq;
using AutoFixture;
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using System.Globalization;
using AMH.Domain.Models;  // adjust namespaces as per your project
using AMH.AFSDK.Services;
using AMH.AFSDK.Repositories;

public class AssetServiceTests
{
    private readonly Mock<IAssetRepository> _assetRepositoryMock;
    private readonly Fixture _fixture;
    private readonly AssetService _assetService;

    public AssetServiceTests()
    {
        _fixture = new Fixture();
        _assetRepositoryMock = new Mock<IAssetRepository>();

        _assetService = new AssetService(_assetRepositoryMock.Object);
    }

    [Fact]
    public async Task GetKPITooltipBySapIdAsync_ShouldReturnValidResponse_WhenValidDataProvided()
    {
        // Arrange
        var request = _fixture.Build<KpiInputBySapIdRequest>()
                              .With(x => x.sapId, _fixture.Create<string>())
                              .With(x => x.kpiCode, _fixture.Create<string>())
                              .With(x => x.startDate, _fixture.Create<DateTime>().ToString("yyyy-MM-dd", CultureInfo.InvariantCulture))
                              .With(x => x.endDate, _fixture.Create<DateTime>().ToString("yyyy-MM-dd", CultureInfo.InvariantCulture))
                              .Create();

        var sqlDataList = _fixture.Build<KpiTooltipDetails>()
                                  .With(x => x.kpiUom, _fixture.Create<string>())
                                  .With(x => x.kpiName, _fixture.Create<string>())
                                  .With(x => x.kpiDefinition, _fixture.Create<string>())
                                  .With(x => x.kpiFormula, _fixture.Create<string>())
                                  .With(x => x.kpiPurpose, _fixture.Create<string>())
                                  .With(x => x.description, _fixture.Create<string>())
                                  .CreateMany(1).ToList();

        var fakeRow = new Dictionary<string, object?>
        {
            { "absolute", _fixture.Create<decimal>() },
            { "orders", _fixture.Create<int>() }
        };

        var fakeQueryResult = new List<dynamic> { fakeRow };

        _assetRepositoryMock.Setup(x => x.GetKPITooltipDetailFromSqlDBAsync(It.IsAny<KpiInputBySapIdRequest>()))
                            .ReturnsAsync(sqlDataList);

        _assetRepositoryMock.Setup(x => x.ExecuteBigDataQueryAsync<dynamic>(
                It.IsAny<string>(),
                It.IsAny<Func<System.Data.Common.DbDataReader, dynamic>>()))
            .ReturnsAsync(fakeQueryResult);

        // Act
        var result = await _assetService.GetKPITooltipBySapIdAsync(request, request.startDate!, request.endDate!);

        // Assert
        Assert.NotNull(result);
        Assert.NotNull(result.labels);
        Assert.NotEmpty(result.labels);
        Assert.All(result.labels, label =>
        {
            Assert.False(string.IsNullOrWhiteSpace(label.title));
            Assert.False(string.IsNullOrWhiteSpace(label.value));
        });

        Assert.NotNull(result.kpiTooltipDetails);
        Assert.Equal(sqlDataList.First().kpiName, result.kpiTooltipDetails.kpiName);
        Assert.Equal(sqlDataList.First().kpiUom, result.kpiTooltipDetails.kpiUom);
    }
}
