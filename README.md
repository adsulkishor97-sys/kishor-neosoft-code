 [Fact]
    public async Task GetTotalCostOfOwnershipByPlantId_ReturnsOk_WhenDataExists()
    {
        // Arrange
        var request = new GetTotalCostOfOwnershipRequestByplantId
        {
            plantId = 10,
            startDate = "2025-01-01",
            endDate = "2025-01-31",
            affiliateId = 5
        };

        var expectedResponse = new GetTotalCostOfOwnershipResponse
        {
            maintenance = 1200,
            operation = 800,
            total = 2000
        };

        _currentServiceMock
            .Setup(s => s.GetTotalCostOfOwnershipByPlantIdAsync(It.IsAny<GetTotalCostOfOwnershipRequestByplantId>()))
            .ReturnsAsync(expectedResponse);

        // Act
        var result = await _controller.GetTotalCostOfOwnershipByPlantId(request);

        // Assert
        var okResult = Assert.IsType<OkObjectResult>(result);
        var actualResponse = Assert.IsType<GetTotalCostOfOwnershipResponse>(okResult.Value);
        Assert.Equal(expectedResponse.total, actualResponse.total);
    }

    // ✅ Test 2: Should return NoContent when service returns null
    [Fact]
    public async Task GetTotalCostOfOwnershipByPlantId_ReturnsNoContent_WhenResponseIsNull()
    {
        // Arrange
        var request = new GetTotalCostOfOwnershipRequestByplantId
        {
            plantId = 20,
            startDate = "2025-01-01",
            endDate = "2025-01-31"
        };

        _currentServiceMock
            .Setup(s => s.GetTotalCostOfOwnershipByPlantIdAsync(It.IsAny<GetTotalCostOfOwnershipRequestByplantId>()))
            .ReturnsAsync((GetTotalCostOfOwnershipResponse?)null);

        // Act
        var result = await _controller.GetTotalCostOfOwnershipByPlantId(request);

        // Assert
        Assert.IsType<NoContentResult>(result);
    }

    // ✅ Optional Test 3: Handle exceptions gracefully (for full coverage)
    [Fact]
    public async Task GetTotalCostOfOwnershipByPlantId_ThrowsException_ShouldPropagate()
    {
        // Arrange
        var request = new GetTotalCostOfOwnershipRequestByplantId { plantId = 30 };

        _currentServiceMock
            .Setup(s => s.GetTotalCostOfOwnershipByPlantIdAsync(It.IsAny<GetTotalCostOfOwnershipRequestByplantId>()))
            .ThrowsAsync(new System.Exception("Database error"));

        // Act & Assert
        await Assert.ThrowsAsync<System.Exception>(() => _controller.GetTotalCostOfOwnershipByPlantId(request));
    }
