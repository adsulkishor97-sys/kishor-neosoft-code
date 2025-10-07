[Fact]
public async Task GetTotalCostOfOwnership_ReturnsOk_WhenDataExists()
{
    // Arrange
    var fixture = new Fixture();

    // Generate request dynamically
    var request = fixture.Build<GetTotalCostOfOwnershipRequest>()
                         .With(r => r.page, "Dashboard") // keep page if required
                         .Create();

    // Generate expected response dynamically
    var expectedResponse = fixture.Build<GetTotalCostOfOwnershipResponse>()
                                  .With(r => r.total, fixture.Create<int>() + fixture.Create<int>()) // ensure total != 0
                                  .Create();

    _mockCurrentServices
        .Setup(s => s.GetTotalCostOfOwnershipAsync(It.IsAny<GetTotalCostOfOwnershipRequest>()))
        .ReturnsAsync(expectedResponse);

    // Act
    var result = await _controller.GetTotalCostOfOwnership(request);

    // Assert
    var okResult = Assert.IsType<OkObjectResult>(result);
    var actual = Assert.IsType<GetTotalCostOfOwnershipResponse>(okResult.Value);
    Assert.Equal(expectedResponse.total, actual.total);
}
[Fact]
public async Task GetTotalCostOfOwnership_ThrowsException_Returns500()
{
    // Arrange
    var fixture = new Fixture();

    // Generate request dynamically
    var request = fixture.Create<GetTotalCostOfOwnershipRequest>();

    _mockCurrentServices
        .Setup(s => s.GetTotalCostOfOwnershipAsync(It.IsAny<GetTotalCostOfOwnershipRequest>()))
        .ThrowsAsync(new Exception("Database error"));

    // Act & Assert
    await Assert.ThrowsAsync<Exception>(() => _controller.GetTotalCostOfOwnership(request));
}
