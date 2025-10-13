[Fact]
    public async Task GetBenchmarkDesignBySapId_ReturnsOk_WhenDataExists()
    {
        // Arrange
        var request = _fixture.Create<GetBenchmarkDesignBySapIdRequest>();
        var expectedResponse = _fixture.CreateMany<GetBenchmarkDesignBySapIdResponse>(3).ToList();

        _mockBenchmarkServices
            .Setup(s => s.GetBenchmarkDesignBySapId(It.IsAny<GetBenchmarkDesignBySapIdRequest>()))
            .ReturnsAsync(expectedResponse);

        // Act
        var result = await _controller.GetBenchmarkDesignBySapId(request);

        // Assert
        var okResult = Assert.IsType<OkObjectResult>(result);
        var actual = Assert.IsType<List<GetBenchmarkDesignBySapIdResponse>>(okResult.Value);
        Assert.Equal(expectedResponse.Count, actual.Count);
    }

    [Fact]
    public async Task GetBenchmarkDesignBySapId_ReturnsNoContent_WhenServiceReturnsNull()
    {
        // Arrange
        var request = _fixture.Create<GetBenchmarkDesignBySapIdRequest>();

        _mockBenchmarkServices
            .Setup(s => s.GetBenchmarkDesignBySapId(It.IsAny<GetBenchmarkDesignBySapIdRequest>()))
            .ReturnsAsync((List<GetBenchmarkDesignBySapIdResponse>?)null);

        // Act
        var result = await _controller.GetBenchmarkDesignBySapId(request);

        // Assert
        Assert.IsType<NoContentResult>(result);
    }

    [Fact]
    public async Task GetBenchmarkDesignBySapId_ThrowsException_ShouldPropagate()
    {
        // Arrange
        var request = _fixture.Create<GetBenchmarkDesignBySapIdRequest>();
        var exceptionMessage = _fixture.Create<string>();

        _mockBenchmarkServices
            .Setup(s => s.GetBenchmarkDesignBySapId(It.IsAny<GetBenchmarkDesignBySapIdRequest>()))
            .ThrowsAsync(new System.Exception(exceptionMessage));

        // Act & Assert
        await Assert.ThrowsAsync<System.Exception>(() => _controller.GetBenchmarkDesignBySapId(request));
    }
