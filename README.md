 [Fact]
    public async Task HandleAffiliateDistributionGlobalPageRequest_ShouldReturnExpectedResult_WhenValidInput()
    {
        // Arrange
        var request = _fixture.Build<GetAffiliatePlantDistributionRequest>()
                              .With(x => x.kpiCode, _fixture.Create<string>())
                              .Create();

        var affiliateRequest = _fixture.Create<string>();
        var formattedStartDate = _fixture.Create<string>();
        var formattedEndDate = _fixture.Create<string>();
        var quotedPmCodes = _fixture.Create<string>();

        var service = new CurrentServices(
            _mockCurrentRepository.Object,
            _mockConfiguration.Object
        );

        // --- Access the private method via reflection ---
        var methodInfo = typeof(CurrentServices).GetMethod(
            "HandleAffiliateDistributionGlobalPageRequest",
            BindingFlags.NonPublic | BindingFlags.Instance);

        Assert.NotNull(methodInfo);

        // --- Act ---
        var resultTask = (Task<List<GroupedData>>)methodInfo.Invoke(service, new object[]
        {
            request,
            affiliateRequest,
            formattedStartDate,
            formattedEndDate,
            quotedPmCodes
        });

        var result = await resultTask;

        // --- Assert ---
        Assert.NotNull(result);
        Assert.IsType<List<GroupedData>>(result);
    }
