[Fact]
public async Task GetTotalCostOfOwnershipByPlantId_ReturnsOk_WhenDataExists()
{
    // Arrange
    var fixture = new Fixture();

    // Generate request dynamically
    var request = fixture.Create<GetTotalCostOfOwnershipRequestByplantId>();

    // Generate expected response dynamically
    var expectedResponse = fixture.Build<GetTotalCostOfOwnershipResponse>()
                                  .With(r => r.total, fixture.Create<int>() + fixture.Create<int>()) // ensure total is positive
                                  .Create();

    _mockCurrentServices
        .Setup(s => s.GetTotalCostOfOwnershipByPlantIdAsync(It.IsAny<GetTotalCostOfOwnershipRequestByplantId>()))
        .ReturnsAsync(expectedResponse);

    // Act
    var result = await _controller.GetTotalCostOfOwnershipByPlantId(request);

    // Assert
    var okResult = Assert.IsType<OkObjectResult>(result);
    var actualResponse = Assert.IsType<GetTotalCostOfOwnershipResponse>(okResult.Value);
    Assert.Equal(expectedResponse.total, actualResponse.total);
}
[Fact]
public async Task GetTotalCostOfOwnershipByPlantId_ThrowsException_ShouldPropagate()
{
    // Arrange
    var fixture = new Fixture();

    // Generate request dynamically
    var request = fixture.Create<GetTotalCostOfOwnershipRequestByplantId>();

    _mockCurrentServices
        .Setup(s => s.GetTotalCostOfOwnershipByPlantIdAsync(It.IsAny<GetTotalCostOfOwnershipRequestByplantId>()))
        .ThrowsAsync(new Exception("Database error"));

    // Act & Assert
    await Assert.ThrowsAsync<Exception>(() => _controller.GetTotalCostOfOwnershipByPlantId(request));
}
