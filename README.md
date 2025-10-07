 [Fact]
 public async Task GetKPITooltip_ReturnsOk_WhenDataExists()
 {
     // Arrange
     var request = new KpiInputRequest
     {
         affiliateId = 1,
         kpiCode = "KPI001",
         startDate = "2025-01-01",
         endDate = "2025-02-01"
     };

     var affiliateCodes = new List<int> { 1001 };
     _mockConfigServices.Setup(s => s.GetAffiliateCodeList(It.IsAny<int?>()))
         .Returns(affiliateCodes);

     var mockResponse = new KpiTooltipResponse
     {
         labels = new List<Label>
 {
     new Label { title = "Total", uom = "kg", value = "100" }
 },
         kpiTooltipDetails = new KpiTooltipDetails()
     };

     _mockCurrentServices.Setup(s => s.GetKPITooltipAsyncNew(
         It.IsAny<KpiInputRequest>(),
         It.IsAny<string>(),
         It.IsAny<string>(),
         It.IsAny<string>()
     )).ReturnsAsync(mockResponse);

     // Act
     var result = await _controller.GetKPITooltip(request);

     // Assert
     var okResult = Assert.IsType<OkObjectResult>(result);
     var value = Assert.IsType<KpiTooltipResponse>(okResult.Value);
     Assert.NotNull(value.labels);
 }

 [Fact]
 public async Task GetAffiliatePlantDistribution_ReturnsOk_WhenDataExists()
 {
     // Arrange
     var request = new GetAffiliatePlantDistributionRequest
     {
         affiliateId = 1,
         page = "Dashboard",
         startDate = "2025-01-01",
         endDate = "2025-02-01"
     };

     var affiliateCodes = new List<int> { 11, 12 };
     _mockConfigServices.Setup(s => s.GetAffiliateCodeList(It.IsAny<int?>()))
         .Returns(affiliateCodes);

     var expectedData = new List<GroupedData>
 {
     new GroupedData { affiliateId = "1", name = "Plant A", overall = 90 },
     new GroupedData { affiliateId = "2", name = "Plant B", overall = 75 }
 };

     _mockCurrentServices.Setup(s => s.GetAffiliatesDistributionAsyncNew(
         It.IsAny<GetAffiliatePlantDistributionRequest>(),
         It.IsAny<string>(),
         It.IsAny<int?>()
     )).ReturnsAsync(expectedData);

     // Act
     var result = await _controller.GetAffiliatePlantDistribution(request);

     // Assert
     var okResult = Assert.IsType<OkObjectResult>(result);
     var data = Assert.IsType<List<GroupedData>>(okResult.Value);
     Assert.Equal(2, data.Count);
 }


 
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
     var request = new GetTotalCostOfOwnershipRequest
     {
         affiliateId = 200
     };

     _mockCurrentServices
         .Setup(s => s.GetTotalCostOfOwnershipAsync(It.IsAny<GetTotalCostOfOwnershipRequest>()))
         .ThrowsAsync(new System.Exception("Database error"));

     // Act & Assert
     await Assert.ThrowsAsync<System.Exception>(() => _controller.GetTotalCostOfOwnership(request));
 }

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
     var request = new GetTotalCostOfOwnershipRequestByplantId { plantId = 30 };

     _mockCurrentServices
         .Setup(s => s.GetTotalCostOfOwnershipByPlantIdAsync(It.IsAny<GetTotalCostOfOwnershipRequestByplantId>()))
         .ThrowsAsync(new System.Exception("Database error"));

     // Act & Assert
     await Assert.ThrowsAsync<System.Exception>(() => _controller.GetTotalCostOfOwnershipByPlantId(request));
 }

 [Fact]
 public async Task GetAssetDistribution_ReturnsOk_WhenDataExists()
 {
     // Arrange
     var request = new GetAssetDistributionRequest
     {
         affiliateId = 1,
         page = "global"
     };

     var affiliateCodes = new List<int> { 10, 20 };
     var expectedResult = new List<AssetGroupedData>
 {
     new AssetGroupedData { affiliateId = "10", name = "Plant A", overall = 100 },
     new AssetGroupedData { affiliateId = "20", name = "Plant B", overall = 200 }
 };

     _mockConfigServices.Setup(x => x.GetAffiliateCodeList(request.affiliateId)).Returns(affiliateCodes);

     _mockCurrentServices
         .Setup(x => x.GetAssetDistributionAsync(It.IsAny<GetAssetDistributionRequest>(), It.IsAny<string>()))
         .ReturnsAsync(expectedResult);

     // Act
     var result = await _controller.GetAssetDistribution(request);

     // Assert
     var okResult = Assert.IsType<OkObjectResult>(result);
     var response = Assert.IsAssignableFrom<List<AssetGroupedData>>(okResult.Value);
     Assert.Equal(2, response.Count);
     Assert.Equal("Plant A", response[0].name);
 }

 [Fact]
 public async Task GetAssetDistribution_ThrowsException_ShouldPropagate()
 {
     // Arrange
     var request = new GetAssetDistributionRequest { affiliateId = 4 };

     _mockConfigServices.Setup(x => x.GetAffiliateCodeList(request.affiliateId))
         .Throws(new System.Exception("DB connection failed"));

     // Act & Assert
     await Assert.ThrowsAsync<System.Exception>(() => _controller.GetAssetDistribution(request));
 }
