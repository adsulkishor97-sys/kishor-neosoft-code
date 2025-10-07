// ✅ Test Case 1: Returns Ok when data is found
    [Fact]
    public async Task GetTotalCostOfOwnership_ReturnsOk_WhenDataExists()
    {
        // Arrange
        var request = new GetTotalCostOfOwnershipRequest
        {
            affiliateId = 101,
            startDate = "2025-01-01",
            endDate = "2025-02-01",
            page = "Dashboard"
        };

        var expectedResponse = new GetTotalCostOfOwnershipResponse
        {
            maintenance = 1000,
            operation = 500,
            total = 1500
        };

        _currentServiceMock
            .Setup(s => s.GetTotalCostOfOwnershipAsync(It.IsAny<GetTotalCostOfOwnershipRequest>()))
            .ReturnsAsync(expectedResponse);

        // Act
        var result = await _controller.GetTotalCostOfOwnership(request);

        // Assert
        var okResult = Assert.IsType<OkObjectResult>(result);
        var actual = Assert.IsType<GetTotalCostOfOwnershipResponse>(okResult.Value);
        Assert.Equal(expectedResponse.total, actual.total);
    }

    // ✅ Test Case 2: Returns NoContent when data is null
    [Fact]
    public async Task GetTotalCostOfOwnership_ReturnsNoContent_WhenResponseIsNull()
    {
        // Arrange
        var request = new GetTotalCostOfOwnershipRequest
        {
            affiliateId = 102,
            startDate = "2025-01-01",
            endDate = "2025-02-01"
        };

        _currentServiceMock
            .Setup(s => s.GetTotalCostOfOwnershipAsync(It.IsAny<GetTotalCostOfOwnershipRequest>()))
            .ReturnsAsync((GetTotalCostOfOwnershipResponse?)null);

        // Act
        var result = await _controller.GetTotalCostOfOwnership(request);

        // Assert
        Assert.IsType<NoContentResult>(result);
    }

    // ✅ Optional: Add one more to cover exception handling (if added later)
    [Fact]
    public async Task GetTotalCostOfOwnership_ThrowsException_Returns500()
    {
        // Arrange
        var request = new GetTotalCostOfOwnershipRequest
        {
            affiliateId = 200
        };

        _currentServiceMock
            .Setup(s => s.GetTotalCostOfOwnershipAsync(It.IsAny<GetTotalCostOfOwnershipRequest>()))
            .ThrowsAsync(new System.Exception("Database error"));

        // Act & Assert
        await Assert.ThrowsAsync<System.Exception>(() => _controller.GetTotalCostOfOwnership(request));
    }
