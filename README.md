[Fact]
public async Task GetKpiData_ReturnsOk_WhenDataExists()
{
    // Arrange
    var fixture = new Fixture(); // AutoFixture generates random test data

    var request = fixture.Build<GetKpiDetailsRequest>()
        .With(r => r.page, "Dashboard") // set specific fields if needed
        .With(r => r.startDate, "2025-01-01")
        .With(r => r.endDate, "2025-02-01")
        .Create();

    var affiliateCodes = fixture.CreateMany<int>(2).ToList(); // generates a random list like [101, 202]
    var kpiResponse = fixture.Build<GetKpiDataResponse>()
        .With(x => x.kpiCode, "KPI01")
        .With(x => x.kpiName, "Efficiency")
        .CreateMany(1)
        .ToList();

    _mockConfigServices
        .Setup(s => s.GetAffiliateCodeList(It.IsAny<int?>()))
        .Returns(affiliateCodes);

    _mockCurrentServices
        .Setup(s => s.GetKpiDetailsAsync(
            It.IsAny<string>(),
            It.IsAny<string>(),
            It.IsAny<string>(),
            It.IsAny<string>(),
            It.IsAny<string>()
        ))
        .ReturnsAsync(kpiResponse);

    // Act
    var result = await _controller.GetKpiData(request);

    // Assert
    var okResult = Assert.IsType<OkObjectResult>(result);
    var data = Assert.IsAssignableFrom<List<GetKpiDataResponse>>(okResult.Value);
    Assert.NotEmpty(data);
    Assert.Equal("KPI01", data.First().kpiCode);
}
