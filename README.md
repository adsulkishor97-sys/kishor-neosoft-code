[Fact]
public async Task GetAssetFilters_ReturnsOk_WhenDataExists()
{
    // Arrange
    var expectedResponse = _fixture.CreateMany<GetAssetFiltersResponse>(3).ToList();

    _mockBenchmarkServices
        .Setup(s => s.GetAssetFilters())
        .ReturnsAsync(expectedResponse);

    // Act
    var result = await _controller.GetAsseFilters();

    // Assert
    var okResult = Assert.IsType<OkObjectResult>(result);
    var actual = Assert.IsType<List<GetAssetFiltersResponse>>(okResult.Value);
    Assert.Equal(expectedResponse.Count, actual.Count);
}

[Fact]
public async Task GetAssetFilters_ThrowsException_ShouldPropagate()
{
    // Arrange
    var databaseError = _fixture.Create<string>();
    _mockBenchmarkServices
        .Setup(s => s.GetAssetFilters())
        .ThrowsAsync(new System.Exception(databaseError));

    // Act & Assert
    await Assert.ThrowsAsync<System.Exception>(() => _controller.GetAsseFilters());
}
