 [Fact]
 public async Task HandleAffiliateDistributionNonGlobalPageRequest_ShouldReturnGroupedData_WithValidInputs()
 {
     // Arrange

     // Mock the repository
     var mockRepo = new Mock<ICurrentRepository>();

     // Generate input request using AutoFixture
     var request = _fixture.Build<GetAffiliatePlantDistributionRequest>()
                           .With(x => x.affiliateId, _fixture.Create<int>())
                           .With(x => x.kpiCode, _fixture.Create<string>())
                           .Create();

     var affiliateRequest = _fixture.Create<string>();
     var formattedStartDate = _fixture.Create<string>();
     var formattedEndDate = _fixture.Create<string>();
     var quotedPmCodes = _fixture.Create<string>();

     // Mock GetCaseHierarchyAsync to return some plants
     var lstPlants = _fixture.CreateMany<GetCaseHierarchyResponse>(3).ToList();
     mockRepo.Setup(r => r.GetCaseHierarchyAsync(It.IsAny<int>()))
             .ReturnsAsync(lstPlants);

     // Mock ExecuteBigDataQuery_New for numerator
     var numResult = _fixture.CreateMany<KpiNumeratorDenominatorAffiliateDistribution>(3).ToList();
     mockRepo.Setup(r => r.ExecuteBigDataQuery_New<KpiNumeratorDenominatorAffiliateDistribution>(
             It.Is<string>(q => q.Contains("N")),
             It.IsAny<Func<DbDataReader, KpiNumeratorDenominatorAffiliateDistribution>>()))
         .ReturnsAsync(numResult);

     // Mock ExecuteBigDataQuery_New for denominator
     var denResult = _fixture.CreateMany<KpiNumeratorDenominatorAffiliateDistribution>(3).ToList();
     mockRepo.Setup(r => r.ExecuteBigDataQuery_New<KpiNumeratorDenominatorAffiliateDistribution>(
             It.Is<string>(q => q.Contains("D")),
             It.IsAny<Func<DbDataReader, KpiNumeratorDenominatorAffiliateDistribution>>()))
         .ReturnsAsync(denResult);

    

     // Get private method via reflection
     var method = typeof(CurrentServices)
         .GetMethod("HandleAffiliateDistributionNonGlobalPageRequest", BindingFlags.NonPublic | BindingFlags.Instance);

     Assert.NotNull(method);

     // Act: Invoke private async method
     var task = (Task<List<GroupedData>>)method!.Invoke(
         null,
         new object[] { request, affiliateRequest, formattedStartDate, formattedEndDate, quotedPmCodes }
     )!;

     var result = await task;

     // Assert
     Assert.NotNull(result);
     Assert.NotEmpty(result);

     // Ensure the returned data is valid
     Assert.All(result, item =>
     {
         Assert.NotNull(item);
     });

     // Verify repository calls
     mockRepo.Verify(r => r.GetCaseHierarchyAsync(It.IsAny<int>()), Times.Once);
     mockRepo.Verify(r => r.ExecuteBigDataQuery_New<KpiNumeratorDenominatorAffiliateDistribution>(
         It.IsAny<string>(), It.IsAny<Func<DbDataReader, KpiNumeratorDenominatorAffiliateDistribution>>()), Times.AtLeastOnce);
 }
 System.Reflection.TargetException : Non-static method requires a target.
